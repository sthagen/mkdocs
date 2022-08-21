# MkDocs Plugins

A Guide to installing, using and creating MkDocs Plugins

---

## Installing Plugins

Before a plugin can be used, it must be installed on the system. If you are
using a plugin which comes with MkDocs, then it was installed when you installed
MkDocs. However, to install third party plugins, you need to determine the
appropriate package name and install it using `pip`:

    pip install mkdocs-foo-plugin

Once a plugin has been successfully installed, it is ready to use. It just needs
to be [enabled](#using-plugins) in the configuration file. The [MkDocs Plugins]
wiki page has a growing list of plugins that you can install and use.

## Using Plugins

The [`plugins`][config] configuration option should contain a list of plugins to
use when building the site. Each "plugin" must be a string name assigned to the
plugin (see the documentation for a given plugin to determine its "name"). A
plugin listed here must already be [installed](#installing-plugins).

```yaml
plugins:
    - search
```

Some plugins may provide configuration options of their own. If you would like
to set any configuration options, then you can nest a key/value mapping
(`option_name: option value`) of any options that a given plugin supports. Note
that a colon (`:`) must follow the plugin name and then on a new line the option
name and value must be indented and separated by a colon. If you would like to
define multiple options for a single plugin, each option must be defined on a
separate line.

```yaml
plugins:
    - search:
        lang: en
        foo: bar
```

For information regarding the configuration options available for a given plugin,
see that plugin's documentation.

For a list of default plugins and how to override them, see the
[configuration][config] documentation.

## Developing Plugins

Like MkDocs, plugins must be written in Python. It is generally expected that
each plugin would be distributed as a separate Python module, although it is
possible to define multiple plugins in the same module. At a minimum, a MkDocs
Plugin must consist of a [BasePlugin] subclass and an [entry point] which
points to it.

### BasePlugin

A subclass of `mkdocs.plugins.BasePlugin` should define the behavior of the plugin.
The class generally consists of actions to perform on specific events in the build
process as well as a configuration scheme for the plugin.

All `BasePlugin` subclasses contain the following attributes:

#### config_scheme

*   A tuple of configuration validation instances. Each item must consist of a
    two item tuple in which the first item is the string name of the
    configuration option and the second item is an instance of
    `mkdocs.config.config_options.BaseConfigOption` or any of its subclasses.

    For example, the following `config_scheme` defines three configuration options: `foo`, which accepts a string; `bar`, which accepts an integer; and `baz`, which accepts a boolean value.

        class MyPlugin(mkdocs.plugins.BasePlugin):
            config_scheme = (
                ('foo', mkdocs.config.config_options.Type(str, default='a default value')),
                ('bar', mkdocs.config.config_options.Type(int, default=0)),
                ('baz', mkdocs.config.config_options.Type(bool, default=True))
            )

    When the user's configuration is loaded, the above scheme will be used to
    validate the configuration and fill in any defaults for settings not
    provided by the user. The validation classes may be any of the classes
    provided in `mkdocs.config.config_options` or a third party subclass defined
    in the plugin.

    Any settings provided by the user which fail validation or are not defined
    in the `config_scheme` will raise a `mkdocs.config.base.ValidationError`.

#### config

*   A dictionary of configuration options for the plugin, which is populated by
    the `load_config` method after configuration validation has completed. Use
    this attribute to access options provided by the user.

        def on_pre_build(self, config):
            if self.config['bool_option']:
                # implement "bool_option" functionality here...

All `BasePlugin` subclasses contain the following method(s):

#### load_config(options)

*   Loads configuration from a dictionary of options. Returns a tuple of
    `(errors, warnings)`. This method is called by MkDocs during configuration
    validation and should not need to be called by the plugin.

#### on_&lt;event_name&gt;()

*   Optional methods which define the behavior for specific [events]. The plugin
    should define its behavior within these methods. Replace `<event_name>` with
    the actual name of the event. For example, the `pre_build` event would be
    defined in the `on_pre_build` method.

    Most events accept one positional argument and various keyword arguments. It
    is generally expected that the positional argument would be modified (or
    replaced) by the plugin and returned. If nothing is returned (the method
    returns `None`), then the original, unmodified object is used. The keyword
    arguments are simply provided to give context and/or supply data which may
    be used to determine how the positional argument should be modified. It is
    good practice to accept keyword arguments as `**kwargs`. In the event that
    additional keywords are provided to an event in a future version of MkDocs,
    there will be no need to alter your plugin.

    For example, the following event would add an additional static_template to
    the theme config:

        class MyPlugin(BasePlugin):
            def on_config(self, config, **kwargs):
                config['theme'].static_templates.add('my_template.html')
                return config

### Events

There are three kinds of events: [Global Events], [Page Events] and
[Template Events].

<details class="card">
  <summary class="card-header">
    See a diagram with relations between all the plugin events
  </summary>
  <div class="card-body">
    <ul>
      <li>The events themselves are shown in yellow, with their parameters.
      <li>Arrows show the flow of arguments and outputs of each event.
          Sometimes they're omitted.
      <li>The events are chronologically ordered from top to bottom.
      <li>Dotted lines appear at splits from global events to per-page events.
      <li>Click the events' titles to jump to their description.
    </ul>
--8<-- "docs/img/plugin-events.svg"
  </div>
</details>
<br>

#### Global Events

Global events are called once per build at either the beginning or end of the
build process. Any changes made in these events will have a global effect on the
entire site.

##### on_serve

::: mkdocs.plugins.BasePlugin.on_serve
    options:
        show_root_heading: false

##### on_config

::: mkdocs.plugins.BasePlugin.on_config
    options:
        show_root_heading: false

##### on_pre_build

::: mkdocs.plugins.BasePlugin.on_pre_build
    options:
        show_root_heading: false

##### on_files

::: mkdocs.plugins.BasePlugin.on_files
    options:
        show_root_heading: false

##### on_nav

::: mkdocs.plugins.BasePlugin.on_nav
    options:
        show_root_heading: false

##### on_env

::: mkdocs.plugins.BasePlugin.on_env
    options:
        show_root_heading: false

##### on_post_build

::: mkdocs.plugins.BasePlugin.on_post_build
    options:
        show_root_heading: false

##### on_build_error

::: mkdocs.plugins.BasePlugin.on_build_error
    options:
        show_root_heading: false

#### Template Events

Template events are called once for each non-page template. Each template event
will be called for each template defined in the [extra_templates] config setting
as well as any [static_templates] defined in the theme. All template events are
called after the [env] event and before any [page events].

##### on_pre_template

::: mkdocs.plugins.BasePlugin.on_pre_template
    options:
        show_root_heading: false

##### on_template_context

::: mkdocs.plugins.BasePlugin.on_template_context
    options:
        show_root_heading: false

##### on_post_template

::: mkdocs.plugins.BasePlugin.on_post_template
    options:
        show_root_heading: false

#### Page Events

Page events are called once for each Markdown page included in the site. All
page events are called after the [post_template] event and before the
[post_build] event.

##### on_pre_page

::: mkdocs.plugins.BasePlugin.on_pre_page
    options:
        show_root_heading: false

##### on_page_read_source

::: mkdocs.plugins.BasePlugin.on_page_read_source
    options:
        show_root_heading: false

##### on_page_markdown

::: mkdocs.plugins.BasePlugin.on_page_markdown
    options:
        show_root_heading: false

##### on_page_content

::: mkdocs.plugins.BasePlugin.on_page_content
    options:
        show_root_heading: false

##### on_page_context

::: mkdocs.plugins.BasePlugin.on_page_context
    options:
        show_root_heading: false

##### on_post_page

::: mkdocs.plugins.BasePlugin.on_post_page
    options:
        show_root_heading: false

### Handling Errors

MkDocs defines four error types:

#### `mkdocs.exceptions.MkDocsException`

*   The base class which all MkDocs exceptions inherit from. This should
    not be raised directly. One of the subclasses should be raised instead.

#### `mkdocs.exceptions.ConfigurationError`

*   This error is raised by configuration validation when a validation error
    is encountered. This error should be raised by any configuration options
    defined in a plugin's [config_scheme].

#### `mkdocs.exceptions.BuildError`

*   This error may be raised by MkDocs during the build process. Plugins should
    not raise this error.

#### `mkdocs.exceptions.PluginError`

*   A subclass of `mkdocs.exceptions.BuildError` which can be raised by plugin
    events.

Unexpected and uncaught exceptions will interrupt the build process and produce
typical Python tracebacks, which are useful for debugging your code. However,
users generally find tracebacks overwhelming and often miss the helpful error
message. Therefore, MkDocs will catch any of the errors listed above, retrieve
the error message, and exit immediately with only the helpful message displayed
to the user.

Therefore, you might want to catch any exceptions within your plugin and raise a
`PluginError`, passing in your own custom-crafted message, so that the build
process is aborted with a helpful message.

The [on_build_error] event will be triggered for any exception.

For example:

```python
from mkdocs.exceptions import PluginError
from mkdocs.plugins import BasePlugin


class MyPlugin(BasePlugin):
    def on_post_page(self, output, page, config, **kwargs):
        try:
            # some code that could throw a KeyError
            ...
        except KeyError as error:
            raise PluginError(str(error))

    def on_build_error(self, error):
        # some code to clean things up
        ...
```

### Entry Point

Plugins need to be packaged as Python libraries (distributed on PyPI separate
from MkDocs) and each must register as a Plugin via a setuptools `entry_points`.
Add the following to your `setup.py` script:

```python
entry_points={
    'mkdocs.plugins': [
        'pluginname = path.to.some_plugin:SomePluginClass',
    ]
}
```

The `pluginname` would be the name used by users (in the config file) and
`path.to.some_plugin:SomePluginClass` would be the importable plugin itself
(`from path.to.some_plugin import SomePluginClass`) where `SomePluginClass` is a
subclass of [BasePlugin] which defines the plugin behavior. Naturally, multiple
Plugin classes could exist in the same module. Simply define each as a separate
entry point.

```python
entry_points={
    'mkdocs.plugins': [
        'featureA = path.to.my_plugins:PluginA',
        'featureB = path.to.my_plugins:PluginB'
    ]
}
```

Note that registering a plugin does not activate it. The user still needs to
tell MkDocs to use it via the config.

[BasePlugin]:#baseplugin
[config]: ../user-guide/configuration.md#plugins
[entry point]: #entry-point
[env]: #on_env
[events]: #events
[extra_templates]: ../user-guide/configuration.md#extra_templates
[Global Events]: #global-events
[Page Events]: #page-events
[post_build]: #on_post_build
[post_template]: #on_post_template
[static_templates]: ../user-guide/configuration.md#static_templates
[Template Events]: #template-events
[MkDocs Plugins]: https://github.com/mkdocs/mkdocs/wiki/MkDocs-Plugins
[on_build_error]: #on_build_error
[config_scheme]: #config_scheme
