# Custom Extract tpls

NOTE: If you want to maintain a custom Teleport project with your own templates so that you can compile your own custom phar or track your project in version control, you should start a new project using the [Teleport boilerplate project](https://github.com/modxcms/teleport-project). This will allow you to maintain your own custom code and update the core Teleport code and artifacts as dictated by your own project requirements.


## Defining a tpl

An Extract tpl is a JSON file that describes how to build a Teleport transport package. It consists of three main properties: a `name`, some package `attributes`, and a collection of one or more `vehicles`.

### Naming the tpl

An Extract tpl's `name` is used when producing the file name of an Extracted transport package. For this reason, you should avoid the `-` or other special characters in the tpl `name`. Use `_` for word separators if you need them.

### Attributes for the package

*available in teleport >=1.2.0*

The `attributes` property provides a way to set the attributes of the transport package that is produced by the Extract. For instance, this can be used to set package attributes used by package management when installing Extras, allowing you to use Teleport as an easy way to create installable transport packages for use within any MODX installation. NOTE however that when producing packages to be used as MODX Extras, you must avoid the use of Teleport-specific transport vehicle classes. Limit the vehicles you define for these packages to those available in xPDO itself (`xPDOObjectVehicle`, `xPDOFileVehicle`, `xPDOScriptVehicle`, or `xPDOTransportVehicle`).

Here is an example use of the `attributes` property that could be used to create an installable Extras package which. This theoretical `test` component located in the MODX `core/components/` directory specifies a dependency on the collections extra at version 3.x (for MODX 2.4 package dependency features) and includes the content of a changelog.txt file located within the component's directory as the `changelog` attribute:

    "attributes": {
        "changelog": {
            "sourceType": "fileContent",
            "source": "{+properties.modx.core_path}components/test/changelog.txt"
        },
        "requires": {
            "collections": "~3.0"
        }
    },

As you can see in this example, individual attributes can get their content from a file available to the Extract by setting the value of the attribute to an object with the properties `sourceType` and `source` where the `sourceType` value is set to `fileContent` and the `source` is the full path to the file to use the content from. Other `sourceType`'s may be supported in the future.

### Describing the Vehicles

The `vehicles` property describes a collection of xPDO transport vehicles that will be included in the package in the order they are described. The vehicles can be core xPDOVehicle derivatives, Teleport-specific derivatives, or even custom derivatives.

A simple example from the `settings.tpl.json` included with Teleport describes two xPDOObject vehicles that will package all System and Context Settings from a MODX installation:

    "vehicles": [
        {
            "vehicle_class": "xPDOObjectVehicle",
            "object": {
                "class": "modSystemSetting",
                "criteria": [
                    "1 = 1"
                ],
                "package": "modx"
            },
            "attributes": {
                "preserve_keys": true,
                "update_object": true
            }
        },
        {
            "vehicle_class": "xPDOObjectVehicle",
            "object": {
                "class": "modContextSetting",
                "criteria": [
                    "1 = 1"
                ],
                "package": "modx"
            },
            "attributes": {
                "preserve_keys": true,
                "update_object": true
            }
        }
    ]

When [Inject](../use/inject.md)ed, the packaged object vehicles will preserve their payload's primary keys and update existing objects in the injection target that match by primary key, as described in the attributes provided for each vehicle in the collection.

Each vehicle consists of an optional `vehicle_package`, a `vehicle_class`, an `object` defining the vehicle payload, and optional `attributes` of the vehicle. The `vehicle_package` should be omitted when using the core xPDOVehicle implementations. The `object` will be unique to each `vehicle_class` implementation.

#### xPDOFileVehicle

* `source`: the absolute path to a file or directory to be packaged
* `target`: a PHP expression that will be `eval()`'d during installation to determine where the file/directory is unpacked

#### xPDOObjectVehicle

* `class`: defines the xPDOObject class to be packaged by the vehicle
* `criteria`: an array or object describing the criteria that will be used to select instances of the specified `class`
* `graph`: defines an object graph to use to package related xPDOObjects
* `graphCriteria`: defines the criteria for filtering related xPDOObjects selected by a `graph`
* `script`: an optional script to be used to create the vehicle or vehicles for this vehicle definition
* `package`: the xPDO package name for the specified `class`

#### xPDOScriptVehicle

* `source`: a script to be executed during installation of the vehicle

