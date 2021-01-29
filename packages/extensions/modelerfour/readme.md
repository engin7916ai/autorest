# AutoRest Modeler Four 

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.


### Autorest plugin configuration
- Please don't edit this section unless you're re-configuring how the powershell extension plugs in to AutoRest
AutoRest needs the below config to pick this up as a plug-in - see https://github.com/Azure/autorest/blob/master/docs/developer/architecture/AutoRest-extension.md


### ModelFour Options
You can specify the following options in your configuration for modelerfour:

~~~ markdown
``` yaml
modelerfour: 
  # this will speed up the serialization if you explicitly say you do or do not want yaml tags in the model
  # default - both
  emit-yaml-tags: undefined|true|false

  # this will flatten modelers marked with 'x-ms-client-flatten' 
  # defaults to false if not specified
  flatten-models: false|true    

  # this will flatten parameters when payload-flattening-threshold is specified (or marked in the input spec)
  # defaults to false if not specified
  flatten-payloads: false|true  

  # setting this to false will skip parameter flattening 
  # for operations that have multiple requests (ie, JSON and BINARY)
  multiple-request-parameter-flattening: true|false
  
  # this runs a pre-namer step to clean up names 
  # defaults to true if not specified
  prenamer: true|false          

  # relaxes schema duplication checks to allow schemas with the
  # same name and renames duplicate schema names with a suffix
  # of "AutoGenerated" with an additional numeric suffix if more
  # than 2 duplicates of the same name are detected.
  #
  # defaults to false if not specified.
  #
  # NOTE: This parameter is a temporary workaround and will be
  # removed in a future release!
  lenient-model-deduplication: false|true

  # does a check to see if names in schemas/enums/etc will collide
  # off by default 
  resolve-schema-name-collisons: false|true

  # if you want to keep the flattened models even if they are not used
  # off by default
  keep-unused-flattened-models: false|true

  # merges response headers into response objects 
  # defaults to false if not specified
  # not implemented
  merge-response-headers: false|true 

  # enables parameter grouping via x-ms-parameter-grouping
  # defaults to false if not specified
  group-parameters: false|true

  # some additional sanity checks to help debugging
  # defaults to false
  additional-checks: true|false

  # always create the content-type parameter for binary requests 
  # when it's only one possible value, make it a constant.
  always-create-content-type-parameter: false|true

  # always create the Accept parameter
  always-create-accept-parameter: true|false

  # always create SealedChoiceSchema for x-ms-enum schemas no matter
  # what the settings are.  This can be used to smooth migration from
  # remodeler to modelerfour.
  always-seal-x-ms-enum: false|true

  # In the case where a type only definition is to inherit another type remove it.
  # e.g. ChildSchema: {allOf: [ParentSchema]}. 
  # In this case ChildSchema will be removed and all reference to it will be updated to point to ParentSchema
  remove-empty-child-schemas: false|true

  # customization of the identifier normalization and naming provided by the prenamer.
  # pascal|pascalcase - MultiWordIdentifier 
  # camel|camelcase - multiWordIdentifier 
  # snake|snakecase - multi_word_identifier
  # upper|uppercase - MULTI_WORD_IDENTIFIER 
  # kebab|kebabcase - multi-word-identifier 
  # space|spacecase - spaces between recognized words
  # default is the first one in the list below:
  # you can prefix or postfix a formatted name with + (ie, '_ + camel' or 'pascal + _' )
  naming: 
    preserve-uppercase-max-length: <number> #defaults to 3
    parameter: camel|pascal|snake|upper|kebab|space
    property: camel|pascal|snake|upper|kebab|space
    operation: pascal|camel|snake|upper|kebab|space
    operationGroup:  pascal|camel|snake|upper|kebab|space
    choice:  pascal|camel|snake|upper|kebab|space
    choiceValue:  pascal|camel|snake|upper|kebab|space
    constant:  pascal|camel|snake|upper|kebab|space
    type:  pascal|camel|snake|upper|kebab|space
    client: pascal|camel|snake|upper|kebab|space
    local: _ + camel 
    global: camel    

    override:  # a key/value mapping of names to force to a certain value 
      cmyk : CMYK
      $host: $host
      LRO: LRO

```
~~~

Default options:
```yaml
modelerfour:
  always-create-accept-parameter: true
```

#### ModelerFour

``` yaml 
pipeline-model: v3
modelerfour-loaded: true
```


``` yaml !$(enable-deduplication)
# By default, modeler-four based generators will not use the deduplicator or subset reducer
# if we need to easily disable this set the enable-deduplication flag.
pass-thru:
  - model-deduplicator
  - subset-reducer
```

``` yaml
modelerfour:
  naming:
    override:  # defaults 
      cmyk : CMYK
      $host: $host

pipeline:
  prechecker:
    input: openapi-document/multi-api/identity

  modelerfour:
    input: 
      - prechecker

  modelerfour/new-transform:
    input: modelerfour

  modelerfour/flattener:
    input: modelerfour/new-transform

  modelerfour/flattener/new-transform:
    input: modelerfour/flattener

  modelerfour/grouper:
    input: modelerfour/flattener/new-transform

  modelerfour/grouper/new-transform:
    input: modelerfour/grouper

  modelerfour/pre-namer:
    input: modelerfour/grouper/new-transform

  modelerfour/pre-namer/new-transform:
    input: modelerfour/pre-namer

  modelerfour/checker:
    input: 
      - modelerfour/pre-namer/new-transform
      - prechecker

  modelerfour/identity:
    input: modelerfour/checker

  modelerfour/emitter:
    input: modelerfour/identity
    scope: scope-modelerfour/emitter

  modelerfour/notags/emitter:
    input: modelerfour/identity
    scope: scope-modelerfour/notags/emitter

scope-modelerfour/emitter: # writing to disk settings
  input-artifact: code-model-v4
  is-object: true # tells autorest that it is an object graph instead of a text document
  output-uri-expr: | # forces filename if it gets written to disk.
    "code-model-v4.yaml"  
    
scope-modelerfour/notags/emitter: # writing to disk settings
  input-artifact: code-model-v4-no-tags
  is-object: true # tells autorest that it is an object graph instead of a text document
  output-uri-expr: | # forces filename if it gets written to disk.
    "code-model-v4-no-tags.yaml"  

# the default preference for modeler-four based generators is to deduplicate inline models fully.
# this may impact performance on extremely large models with a lot of inline schemas.
deduplicate-inline-models: true

```

``` yaml $(inspector) 
pipeline:
  inspector/codemodel/reset-identity:
    input: 
      - prechecker
      - modelerfour/identity
      - inspector

    to: inspect-document
  
  inspector/emitter:
    input: 
      - inspector/codemodel/reset-identity    
```