---
title: Ditch the Template Chaos - Angular Dynamic Components Loading
subtitle: Preventing Eye Cancer from HTML Pollution
domain: software-engineering-corner.hashnode.dev
tags: web-development, angular, components, html, template
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/9nXBuHKkafU/upload/32547a47488e49867018f2436a958559.jpeg?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp
publishAs: timouti
hideFromHashnodeCommunity: false
ignorePost: true
---

Sometimes you have to deal with handling various views for different data types. In this article, we explore dynamic components loading in Angular.

### Problem
Let's say we are fetching a list of cameras and camera equipment for our glorious online shop. Each item is either a camera or camera equipment. You know, like tripods, lenses, bags, and whatever fancy filters come to your mind. All those items share some common attributes like price, name, some overselling description, and an id. However, when it comes to the technical characteristics, they differ. For cameras, we want to show the list of tech attributes in a table. And for the equipment, we ought to use single values, references, or whatever fancy displaying alternative comes to your mind. The point is, that we need to employ different views depending on the item type.

### Dynamic Components Loading
Now, we already know that we need to create different view components. So, one view for the tech spec table, one for simple string lists, etc. You get the drill. The question remains how to easily switch between those view components depending on the data type of our attributes. Traditionally, you would use an `ngIf` or `ngSwitch` for such a task (or factory resolvers if you are old-school). However, you might need to handle more than just four views. In that case, both approaches don't scale well. There is enough chaos and pollution in the world. So, we want to keep at least our template nice and clean.

Luckily, there is another powerful directive that is rarely used: `ngComponentOutlet` (I know, it might be a branding issue). This directive allows us to load new components dynamically at runtime. Additionally, as of Angular 16.2, we can provide inputs to those dynamically created components.

## Example
Let's see it in action. We loaded our list of camera/equipment items from the backend. We already created various view components for different attribute types, like a table, single line value, a reference to other cameras, or a table of references to cameras (e.g. for compatibility). We have a common `ItemComponent` that displays the name, price, description, etc. This is the place where we want to add our component outlet.

In our template, we add the `ngComponentOutlet` and pass it the view component and inputs, required for displaying. Admittedly, the syntax feels a bit strange. 

```html
<ng-container 
        *ngComponentOutlet="getAttributeViewComponent(item); 
            inputs: {
                attributes: item.attributes
            }"
/>
```

In the `ItemComponent`, we have to implement the `getAttributeViewComponent` function. It should return the desired view component based on the data type.

```typescript
getAttributeViewComponent(item: ShopItem) {
    switch (item.attributeType) {
        case AttributeType.LIST:
            return ListViewComponent;
        case AttributeType.TABLE:
            return TableViewComponent;
        case AttributeType.REFERENCE:
            return ReferenceViewComponent;
        case AttributeType.REFERENCE_TABLE:
            return ReferenceTableViewComponent;
        default:
            return SingleValueViewComponent;
    }
}
```

That's it. The attribute views are now being loaded dynamically at run time based on the data type. Lean, isn't it?

## Conclusion
The new input property of `ngComponentOutlet` allows us to use the directive more flexibly and easily. This results in our template being clean and tidy. Just the way we like it.