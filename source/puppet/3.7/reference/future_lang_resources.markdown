---
layout: default
title: "Future Parser: Resources"
canonical: "/puppet/latest/reference/future_lang_resources.html"
---

[realize]: ./future_lang_virtual.html#syntax
[virtual]: ./future_lang_virtual.html
[containment]: ./future_lang_containment.html
[scope]: ./future_lang_scope.html
[report]: /guides/reporting.html
[append_attributes]: ./future_lang_classes.html#appending-to-resource-attributes
[types]: /references/3.7.latest/type.html
[string]: ./future_lang_data_string.html
[array]: ./future_lang_data_array.html
[datatype]: ./future_lang_data.html
[inheritance]: ./future_lang_classes.html#inheritance
[relationships]: ./future_lang_relationships.html
[resdefaults]: ./future_lang_defaults.html
[reference]: ./future_lang_data_resource_reference.html
[class]: ./future_lang_classes.html
[defined_type]: ./future_lang_defined_types.html
[collector]: ./future_lang_collectors.html
[catalog]: ./future_lang_summary.html#compilation-and-catalogs

> * [See the resource type reference for complete information about Puppet's built-in resource types.][types]

**Resources** are the fundamental unit for modeling system configurations. Each resource describes some aspect of a system, like a specific service or package.

A **resource declaration** is an expression in the Puppet language which describes a resource and tells Puppet to add it to the [catalog][].

When Puppet applies a catalog to a target system, it manages all the resources included in the catalog, ensuring that the actual state matches the desired state.

The Puppet language includes some constructs that let you write a resource declaration but delay adding it to the catalog. For example:

* [Class definitions][class] and [defined types][defined_type] contain resources, but their contents are only added to the catalog once the class or an instance of the defined type is _declared._ They make groups of resources available and let you add the whole group to the catalog as needed.
* [Virtual resources][virtual] are only added to the catalog once they are [realized][realize].

Syntax
-----

{% highlight ruby %}
    # A resource declaration:
    file { '/etc/passwd':
      ensure => file,
      owner  => 'root',
      group  => 'root',
      mode   => '0600',
    }
{% endhighlight %}

Every resource has a **resource type,** a **title,** and a set of **attributes:**

{% highlight ruby %}
    type {'title':
      attribute => value,
    }
{% endhighlight %}

The general form of a resource declaration is:

* The resource type, in lower-case
* An opening curly brace
* The title, which is a [string][]
* A colon
* Optionally, any number of attribute and value pairs, each of which consists of:
    * An attribute name, which is a bare word
    * A `=>` (arrow, fat comma, or hash rocket)
    * A value, which can be any [data type][datatype], depending on what the attribute requires
    * A trailing comma (note that the comma is optional after the final attribute/value pair)
* Optionally, a semicolon, followed by another title, colon, and attribute block
* A closing curly brace

Note that, in the Puppet language, whitespace is fungible.

### Resource Type

The resource type identifies what kind of resource it is. Puppet has a large number of built-in resource types, including files on disk, cron jobs, user accounts, services, and software packages. [See here for a list of built-in resource types][types].

Puppet can be extended with additional resource types, written in Ruby or in the Puppet language.

### Title

The title is an identifying string. It only has to identify the resource to Puppet's compiler; it does not need to bear any relationship to the actual target system.

Titles **must be unique per resource type.** You may have a package and a service both titled "ntp," but you may only have one service titled "ntp." Duplicate titles will cause a compilation failure.

### Attributes

Attributes describe the desired state of the resource; each attribute handles some aspect of the resource.

Each resource type has its own set of available attributes; see [the resource type reference][types] for a complete list. Most resource types have a handful of crucial attributes and a larger number of optional ones. Many attributes have a default value that will be used if a value isn't specified.

Every attribute you declare must have a value; the [data type][datatype] of the value depends on what the attribute accepts. Most attributes that can take multiple values accept them as an [array][].

> #### Synonym Note: Parameters and Properties
>
> When discussing resources and types, **parameter** is a synonym for attribute. You might also hear **property,** which has a slightly different meaning when discussing the Ruby implementation of a resource type or provider. (Properties always represent concrete state on the target system. A provider can check the current state of a property, and switch it to new states.)
>
> When talking about resource declarations in the Puppet language, you should use either "attribute" or "parameter." We suggest "attribute."

Behavior
-----

A resource declaration adds a resource to the catalog, and tells Puppet to manage that resource's state. When Puppet applies the compiled catalog, it will:

* Read the actual state of the resource on the target system
* Compare the actual state to the desired state
* If necessary, change the system to enforce the desired state

### Uniqueness

Puppet does not allow you to declare the same resource twice. This is to prevent multiple conflicting values from being declared for the same attribute.

Puppet uses the [title](#title) and [name/namevar](#namenamevar) to identify duplicate resources --- if either of these is duplicated within a given resource type, the compilation will fail.

If multiple classes require the same resource, you can use a [class][] or a [virtual resource][virtual] to add it to the catalog in multiple places without duplicating it.

### Relationships and Ordering

[ordering]: /references/3.7.latest/configuration.html#ordering

By default, the order of resources in a manifest doesn't affect the order in which those resources will be applied. Puppet will apply _unrelated_ resources in a mostly random (but consistent between runs) order.

If a resource must be applied before or after some other resource, you should declare a relationship between them, to make sure Puppet applies them in the right order. You can also make changes in one resource cause a refresh of some other resource. See [the Relationships and Ordering page][relationships] for more information.

You can also change [the `ordering` setting][ordering] to make Puppet apply unrelated resources in manifest order. This will be the new default behavior in Puppet 4.

### Changes, Events, and Reporting

If Puppet makes any changes to a resource, it will log those changes as events. These events will appear in Puppet agent's log and in the run [report][], which is sent to the Puppet master and forwarded to any number of report processors.

### Scope Independence

Resources are not subject to [scope][] --- a resource in any scope may be [referenced][reference] from any other scope, and local scopes do not introduce local namespaces for resource titles.

### Containment

Resources may be contained by [classes][class] and [defined types][defined_type] --- when something forms a [relationship][relationships] with the container, the contained resources are also affected. See [Containment][] for more details.

Special Attributes
-----

### Name/Namevar

Most resource types have an attribute which identifies a resource _on the target system._ This special attribute is called the "namevar," and the attribute itself is often (but not always) just `name`. For example, the `name` of a service or package is the name by which the system's service or package tools will recognize it. On the other hand, the `file` type's namevar is `path`, the file's location on disk.

The [resource type reference][types] lists the namevars for all of the core resource types. For custom resource types, check the documentation for the module that provides that resource type.

Namevar values **must be unique per resource type,** with only rare exceptions (such as `exec`).

Namevars are not to be confused with the **title**, which identifies a resource _to Puppet._ However, they often have the same value, since the namevar's value will default to the title if it isn't specified. Thus, the `path` of the file example [above](#syntax) is `/etc/passwd`, even though we didn't include the `path` attribute in the resource declaration.

The separation between title and namevar lets you use a consistently-titled resource to manage something whose name differs by platform. For example, the NTP service might be `ntpd` on Red Hat-derived systems, but `ntp` on Debian and Ubuntu; to accommodate that, you could title the service "ntp," but set its name according to the OS. Other resources could then form relationships to it without worrying that its title will change.

### Ensure

Many resource types have an `ensure` attribute. This generally manages the most important aspect of the resource on the target system --- does the file exist, is the service running or stopped, is the package installed or uninstalled, etc.

Allowed values for `ensure` vary by resource type. Most accept `present` and `absent`, but there may be additional variations. Be sure to check the reference for each resource type you are working with.

### Metaparameters

Some attributes in Puppet can be used with every resource type. These are called **metaparameters.** They don't map directly to system state; instead, they specify how Puppet should act toward the resource.

The most commonly used metaparameters are for specifying [order relationships][relationships] between resources.

You can see the full list of all metaparameters in the [Metaparameter Reference](/references/3.7.latest/metaparameter.html).


Condensed Forms
-----

There are two ways to compress multiple resource declarations. You can also use [resource default statements][resdefaults] to reduce duplicate typing.

### Array of Titles

If you specify an array of strings as the title of a resource declaration, Puppet will treat it as multiple resource declarations with an identical block of attributes.

{% highlight ruby %}
    file { ['/etc',
            '/etc/rc.d',
            '/etc/rc.d/init.d',
            '/etc/rc.d/rc0.d',
            '/etc/rc.d/rc1.d',
            '/etc/rc.d/rc2.d',
            '/etc/rc.d/rc3.d',
            '/etc/rc.d/rc4.d',
            '/etc/rc.d/rc5.d',
            '/etc/rc.d/rc6.d']:
      ensure => directory,
      owner  => 'root',
      group  => 'root',
      mode   => '0755',
    }
{% endhighlight %}

This example is the same as declaring each directory as a separate resource with the same attribute block. You can also store an array in a variable and specify the variable as a resource title:

{% highlight ruby %}
    $rcdirectories = ['/etc',
                      '/etc/rc.d',
                      '/etc/rc.d/init.d',
                      '/etc/rc.d/rc0.d',
                      '/etc/rc.d/rc1.d',
                      '/etc/rc.d/rc2.d',
                      '/etc/rc.d/rc3.d',
                      '/etc/rc.d/rc4.d',
                      '/etc/rc.d/rc5.d',
                      '/etc/rc.d/rc6.d']

    file { $rcdirectories:
      ensure => directory,
      owner  => 'root',
      group  => 'root',
      mode   => '0755',
    }
{% endhighlight %}


Note that you cannot specify a separate namevar with an array of titles, since it would then be duplicated across all of the resources. Thus, each title must be a valid namevar value.

### Semicolon After Attribute Block

If you end an attribute block with a semicolon rather than a comma, you may specify another title, another colon, and another complete attribute block, instead of closing the curly braces. Puppet will treat this as multiple resources of a single resource type.

{% highlight ruby %}
    file {
      '/etc/rc.d':
        ensure => directory,
        owner  => 'root',
        group  => 'root',
        mode   => '0755';

      '/etc/rc.d/init.d':
        ensure => directory,
        owner  => 'root',
        group  => 'root',
        mode   => '0755';

      '/etc/rc.d/rc0.d':
        ensure => directory,
        owner  => 'root',
        group  => 'root',
        mode   => '0755';
    }
{% endhighlight %}


Adding or Modifying Attributes
-----

Although you cannot declare the same resource twice, you can add attributes to an already-declared resource. In certain circumstances, you can also override attributes.

### Amending Attributes With a Reference

{% highlight ruby %}
    file {'/etc/passwd':
      ensure => file,
    }

    File['/etc/passwd'] {
      owner => 'root',
      group => 'root',
      mode  => '0640',
    }
{% endhighlight %}

The general form of a reference attribute block is:

* A [reference][] to the resource in question (or a multi-resource reference)
* An opening curly brace
* Any number of attribute => value pairs
* A closing curly brace

In normal circumstances, this idiom can only be used to add previously unmanaged attributes to a resource; it cannot override already-specified attributes. However, within an [inherited class][inheritance], you **can** use this idiom to override attributes.

### Amending Attributes With a Collector

{% highlight ruby %}
    class base::linux {
      file {'/etc/passwd':
        ensure => file,
      }
      ...
    }

    include base::linux

    File <| tag == 'base::linux' |> {
      owner => 'root',
      group => 'root',
      mode  => '0640',
    }
{% endhighlight %}

The general form of a collector attribute block is:

* A [resource collector][collector] that matches any number of resources
* An opening curly brace
* Any number of attribute => value (or attribute +> value) pairs
* A closing curly brace

Much like in an [inherited class][inheritance], you can use the special `+>` keyword to append values to attributes that accept arrays. See [appending to attributes][append_attributes] for more details.

> Note that this idiom **must be used carefully,** if at all:
>
> * It **can always override** already-specified attributes, regardless of class inheritance.
> * It can affect large numbers of resources at once.
> * It will [implicitly realize][realize] any [virtual resources][virtual] that the collector matches. If you are using virtual resources at all, you must use extreme care when constructing collectors that are not intended to realize resources, and would be better off avoiding non-realizing collectors entirely.
> * Since it ignores class inheritance, you can override the same attribute twice, which results in a evaluation-order dependent race where the final override wins.
