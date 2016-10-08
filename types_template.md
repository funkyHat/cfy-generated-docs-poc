---
layout: bt_wiki
title: Google Cloud Plugin
category: Plugins
draft: false
weight: 100
---
{{'{{'}}% gsSummary %{{'}}'}} {{'{{'}}% /gsSummary %{{'}}'}}

The GCP plugin allows users to use Cloudify to manage cloud resources on GCP. See below for currently supported resource types.

Be aware that some services and resources vary in availability between regions and accounts.


# Plugin Requirements

* Python versions:
  * 2.7.x
* [GCP](https://cloud.google.com/) account


# Compatibility

The GCP plugin uses the official [Google API Python Client](https://github.com/google/google-api-python-client).



# Terminology

* Region refers to a general geographical area, such as "Central Europe" or "East US".
* Zone refers to a distinct area within a region. Zones are usually referred to as '{region}-{zone}, i.e. 'us-east1-b' is a zone within the reigon 'us-east1'.


# Types

The following are [node type]({{'{{'}}< relref "blueprints/spec-node-types.md" >{{'}}'}}) definitions. Nodes describe resources in your cloud infrastructure. For more information, see [node type]({{'{{'}}< relref "blueprints/spec-node-types.md" >{{'}}'}}).

### Common Properties

All cloud resource nodes have common properties:

**Properties**

  * `gcp_config` a dictionary that contains values you would like to pass to the connection client. This should be in the format provided by the GCP credentials JSON export (https://developers.google.com/identity/protocols/OAuth2ServiceAccount#creatinganaccount)

Every time you manage a resource with Cloudify,
it creates one or more connections to the GCP API.
You specify the configuration for these clients using the `gcp_config` property.
It should be a dictionary, with the following values:

  * `project` the name of your project on GCP
  * `zone` the default zone which will be used,
    unless overridden by a defined zone/subnetwork
  * `auth` the JSON key file provided by GCP.
    Can either be the contents of the JSON file, or a file path.


Many cloud resources also support the `use_external_resource` and property:

**Properties**

  * `use_external_resource` a boolean for setting whether to create the resource or use an existing one. See the [using existing resources section](#using-existing-resources). Defaults to `false`.
  * `resource_id` The ID of an existing resource when the `use_external_resource` property is set to `true` (see the [using existing resources section](#using-existing-resources)). Defaults to `''` (empty string).



## Runtime Properties

See section on [runtime properties](http://cloudify-plugins-common.readthedocs.org/en/3.3/context.html?highlight=runtime#cloudify.context.NodeInstanceContext.runtime_properties)

Most node types will write a snapshot of the `resource` information from GCP when the node creation has finished (some, e.g. DNSRecord don't correspond directly to an entity in GCP, so this is not universal).


# Node Types
{% for name, content in types.items()|sort %}
## {{ name }}
**Derived From:** {{ content.derived_from_text }}

{{ content['description'] or 'TODO: description' }}


**Properties:**

{% for property, details in content['properties'].items() %}
  * `{{ property }}`
    {{ details.get('description', '')|indent }}
{%- if details['type'] is defined %}
        **type:** {{ details.type }}
{%- endif %}
{%- if details['default'] is defined %}
        **default:** {{details.default}}
{%- elif details.get('required', True) != None %}
        **required** {{ details.get('required') }}
{%- endif %}

{%- endfor %}


{% if content['example'] %}
## Example

{{'{{'}}< gsHighlight  yaml  >{{'}}'}}

{{ content['example']['yaml'] }}
{{'{{'}}< /gsHighlight >{{'}}'}}

{{ content['example']['text'] }}
{% endif %}

{% endfor %}


# Relationships
{% for name, content in relationships.items()|sort %}
## {{ name }}
**Derived From:** {{ content.derived_from }}

{% endfor %}


# Using Existing Resources

It is possible to use existing resources on GCP - whether these have been created by a different Cloudify deployment or not via Cloudify at all.

Many Cloudify GCP types have a property named `use_external_resource`, which defaults to `false`. When set to `true`, the plugin will apply different semantics for each of the operations executed on the relevant node's instances:

This behavior is common to all resource types which support `use_external_resource`:

 * `create` If `use_external_resource` is true, the GCP plugin will check if the resource is available in your account. If no such resource is available, the operation will fail, if it is available, it will assign the resource details to the instance `runtime_properties`.
 * `delete` If `use_external_resource` is true, the GCP plugin will check if the resource is available in your account. If no such resource is available, the operation will fail, if it is available, it will unassign the instance `runtime_properties`.


# Account Information

The plugin needs access to your GCP auth credentials (via the `gcp_config` parameter) in order to operate (but see below about use within a manager).


# Tips

* When packaging blueprints for use with a manager the manager will add the following configurations (you can still override them in your blueprint):
  * `gcp_config`
