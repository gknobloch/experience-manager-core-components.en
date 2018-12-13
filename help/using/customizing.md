---
title: Customizing Core Components
seo-title: Customizing Core Components
description: null
seo-description: The Core Components implement several patterns that allow easy customization, from simple styling to advanced functionality reuse.
uuid: 38d22b85-4867-4716-817a-10ee2f8de6f5
contentOwner: User
content-type: reference
topic-tags: developing
products: SG_EXPERIENCEMANAGER/CORECOMPONENTS-new
discoiquuid: 3c9e0ade-1ce0-4e34-ae04-8da63f9b6c4f
index: y
internal: n
snippet: y
---

# Customizing Core Components{#customizing-core-components}

The [Core Components](../using/developing.md) implement several patterns that allow easy customization, from simple styling to advanced functionality reuse.

## Flexible Architecture {#flexible-architecture}

The Core Components were designed from the beginning to be flexible and extensible. A look at an overview of their architecture reveals where customizations can be made.

![](assets/screen_shot_2018-12-07at093742.png)

* [The design dialog](../using/customizing.md#main-pars_header) defines what authors can or cannot do in the edit dialog.
* [The edit dialog](../using/customizing.md#main-pars_header) shows authors only the options they are allowed to use.
* [The Sling model](../using/customizing.md#main-pars_title_990076047) verifies and prepares the content for the view (template).
* [The result of the Sling model](../using/customizing.md#main-pars_title_990076047) can be serialized to JSON for SPA use-cases.
* [The HTL renders the HTML](../using/customizing.md#main-pars_title_443466850) server-side for traditional server-side rendering.
* [The HTML output](../using/customizing.md#main-pars_title_443466850) is semantic, accessible, search-engine optimized, and easy to style.

And all core components implement the [Style System](../using/customizing.md#main-pars_title_1267131420).

## Customization Patterns {#customization-patterns}

### Customizing Dialogs {#customizing-dialogs}

It may be desired to customize the configuration options available in a core component dialog, be it the [Design Dialog or the Edit Dialog](../using/authoring.md#main-pars_title_1048034172).

Each dialog has a consistent node structure. It is recommended that this structure is replicated in an inheriting component so that [Sling Resource Merger](/content/help/en/experience-manager/6-4/sites/developing/using/sling-resource-merger) and [Hide Conditions](/content/help/en/experience-manager/6-4/sites/developing/using/hide-conditions) can be used to hide, replace, or reorder sections of the original dialog. The structure to replicate is defined as anything up to the tab item node level.

To be fully compatible with any changes made to a dialog on its current version, it is very important that structures below the tab item level not be touched (hidden, added to, replaced, reordered, etc.). Instead, a tab item from the parent should be hidden via the `sling:hideResource` property (see [Sling Resource Merger Properties](/content/help/en/experience-manager/6-4/sites/developing/using/sling-resource-merger)), and new tab items added that contain the bespoke configuration fields. `sling:orderBefore` can be used to reorder tab items if necessary.

The dialog below demonstrates the recommended dialog structure as well as how to hide and replace an inherited tab as described above:

<!-- 

Comment Type: annotation
Last Modified By: ims-author-CE1E2CE451D1F0680A490D45@AdobeID
Last Modified Date: 2017-04-17T17:43:20.265-0400

Should we provide guidance on how to name their CSS classes, etc. to align to component re-usability best-practices? We tout that we follow bootstrap css naming, should we be counseling customers to align similarly? .cmp- 
<component name="">
  -- 
 <element>
   - 
  <element descriptor="">
    ? 
  </element> 
 </element> 
</component>

 -->

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
          xmlns:jcr="http://www.jcp.org/jcr/1.0"
          xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
          xmlns:granite="http://www.adobe.com/jcr/granite/1.0"
          jcr:primaryType="nt:unstructured">
    <content jcr:primaryType="nt:unstructured">
        <items jcr:primaryType="nt:unstructured">
            <tabs jcr:primaryType="nt:unstructured">
                <items jcr:primaryType="nt:unstructured">
                        <originalTab
                                jcr:primaryType="nt:unstructured"
                                sling:hideResource="true"/>
                        </originalTab>
                        <myTab
                               jcr:primaryType="nt:unstructured"
                               jcr:title="My Tab"
                               sling:resourceType="granite/ui/components/coral/foundation/container"/>
                               <!-- myTab content -->
                        </myTab>
                </items>
            </basic>
        </items>
    </content>
</jcr:root>
```

### Customizing the Logic of a Core Component {#customizing-the-logic-of-a-core-component}

The Business logic for the core components is implemented in Sling Models. This logic can be extended by using a Sling delegation pattern.

For example, the title core component uses the `jcr:title` property of the requested resource to provide the title text. If no `jcr:title` property is defined, a fallback to the current page title is implemented. We want to change the behavior so that the title of the current page is always displayed.

Because the implementation of the Core Components' models are private, they must be extended with a delegation pattern.

```java
@Model(adaptables = SlingHttpServletRequest.class,
       adapters = Title.class,
       resourceType = "myproject/components/pageHeadline")
public class PageHeadline implements Title {
    @ScriptVariable private Page currentPage;
    @Self @Via(type = ResourceSuperType.class)
    private Title title;
    @Override public String getText() {
        return currentPage.getTitle();
    }
    @Override public String getType() {
        return title.getType();
    }
}
```

For further details about the delegation pattern see the Core Components GitHub Wiki article [Delegation Pattern for Sling Models](https://github.com/adobe/aem-core-wcm-components/wiki/Delegation-Pattern-for-Sling-Models).

### Customizing the Markup {#customizing-the-markup}

Sometimes advanced styling requires a different markup structure of the component.

This can easily be done by copying the HTL files that need to be modified from the Core Component into the proxy component.

Taking again the example of the Core Breadcrumb Component, to customize its markup output, the `breadcrumb.html` file would have to be copied into the site-specific component that has a `sling:resourceSuperTypes` that points to the Core Breadcrumb Component.

<!-- 

Comment Type: annotation
Last Modified By: ims-author-CE1E2CE451D1F0680A490D45@AdobeID
Last Modified Date: 2017-04-17T17:43:20.265-0400

Should we provide guidance on how to name their CSS classes, etc. to align to component re-usability best-practices? We tout that we follow bootstrap css naming, should we be counseling customers to align similarly? .cmp- 
<component name="">
  -- 
 <element>
   - 
  <element descriptor="">
    ? 
  </element> 
 </element> 
</component>

 -->

### Styling the Components {#styling-the-components}

The first form of customization is to apply CSS styles.

To make this easy, the Core Components render semantic markup and follow a standardized naming convention inspired by [Bootstrap](http://getbootstrap.com/). Also, to easily target and namespace the styles for the individual components, each Core Component is wrapped in a DIV element with the " `cmp`" and " `cmp-<name>`" classes.

For instance, looking at the HTL file of the v1 Core Breadcrumb Component: [breadcrumb.html](https://github.com/adobe/aem-core-wcm-components/blob/master/content/src/content/jcr_root/apps/core/wcm/components/breadcrumb/v2/breadcrumb/breadcrumb.html), we see that the hierarchy of elements output are `ol.breadcrumb > li.breadcrumb-item > a`. So to make sure that a CSS rule only affects the breadcrumb class of that component, all rules should be namespaced as shown below:

`.cmp-breadcrumb .breadcrumb {}  
.cmp-breadcrumb .breadcrumb-item {}  
.cmp-breadcrumb a {}`

Additionally, each of the Core Components leverage the AEM [Style System feature](/content/help/en/experience-manager/6-4/sites/authoring/using/style-system) that allows template authors to define additional CSS class names that can be applied to the component by the page authors. This allows to define for each template a list of allowed component styles, and whether one of them should apply by default to all components of that kind.

## Upgrade Compatibility of Customizations {#upgrade-compatibility-of-customizations}

Three different kind of upgrades are possible:

* upgrading the version of AEM
* upgrading the Core Components to a new minor version
* upgrading the Core Components to a major version

Generally, upgrading AEM to a new version won't impact the Core Components or the customizations done, provided that the components' versions also support the new AEM version that is being migrated to, and that customizations don't use APIs that have been [deprecated or removed](/content/help/en/experience-manager/6-3/release-notes/deprecated-removed-features).

Upgrading the Core Components without switching to a newer major version shouldn't affect customizations, as long as the customization patterns described on this page are used.

Switching to a newer major version of the Core Components is compatible only for the content structure, but customizations may need to be refactored. Clear change logs will be published for each component version to highlight the changes that would impact the kind of customizations described on this page.

## Support of Customizations {#support-of-customizations}

Like for any AEM component, there are a number of things to be aware of regarding customizations:

1. **Never modify the code of Core Components directly.** 
  
   This would make them unsupported entirely, and make future updates of the components a painful process. Instead, use the customization methods described on this page.

1. **Custom code is your own responsibility.** 
  
   Our support program doesn't cover custom code, and reported issues that cannot be reproduced with vanilla Core Components that are [used as documented](../using/using.md) won't qualify.

1. **Watch deprecated and removed functionality.** 
  
   With each new AEM version being upgraded to, ensure that all API used are still topical by keeping an eye on the [Deprecated and Removed Features](/content/help/en/experience-manager/6-3/release-notes/deprecated-removed-features) page.

See also the [Core Component Support](../using/developing.md#CoreComponentSupport) section.

**Read next:**

* [Using Core Components](../using/using.md) - get up-and-running with Core Components in your own project.
* [Component Guidelines](../using/guidelines.md) - to learn the implementation patterns of the Core Components.
