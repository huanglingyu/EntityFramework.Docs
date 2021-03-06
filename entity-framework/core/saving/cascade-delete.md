---
title: Cascade Delete - EF Core
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: ee8e14ec-2158-4c9c-96b5-118715e2ed9e
ms.technology: entity-framework-core

uid: core/saving/cascade-delete
---
# Cascade Delete

Cascade delete is commonly used in database terminology to describe a characteristic that allows the deletion of a row to automatically trigger the deletion of related rows. EF Core implements several different delete behaviors and allows for the configuration of the delete behaviors of individual relationships. EF Core also implements conventions that automatically configure useful default delete behaviors for each relationship based on the [requiredness of the relationship] (../modeling/relationships.md#required-and-optional-relationships).

## Delete behaviors
Delete behaviors are defined in the *DeleteBehavior* enumerator type and can be passed to the *OnDelete* fluent API to  control whether the deletion of a principal/parent entity should have a side effect on dependent/child entities it is related to.

There are four delete behaviors:

| Behavior Name | Effect on dependent entities tracked in memory | Effect on dependent entities stored in database | Used by default for relationships that are
|-|-|-|-  
| **Cascade** | Entities are deleted | Entities are deleted | **Required**  
| **ClientSetNull** | Foreign key properties are set to null | None | **Optional**  
| **SetNull** | Foreign key properties are set to null | Foreign key properties are set to null | None  
| **Restrict** | None | None | None  

> [!IMPORTANT]  
> **Changes in EF Core 2.0:** In previous releases, *Restrict* would cause optional foreign key properties in tracked dependent entities to be set to null, and was the default delete behavior for optional relationships. In EF Core 2.0, the *ClientSetNull* was introduced to represent that behavior and became the default for optional relationships. The behavior of *Restrict* was adjusted to never have any side effects on dependent entities.

When either *ClientSetNull* or *SetNull* are explicitly configured on a required relationship (i.e. a relationship controlled by a non-nullable foreign key property), if the principal/parent is deleted and any dependent/child entity exist, the foreign key property won't be set to null (because it cannot be set to null), but instead the attempt to set it to null will be recorded and *SaveChanges* will fail, unless the dependent/child eneities are moved to a new principal/parent.

> [!NOTE]  
> The delete behavior configured in the EF Core model is only applied when the principal entity is deleted using EF Core and the dependent entities are loaded in memory (i.e. for tracked dependents). A corresponding cascade behavior needs to be setup in the database to ensure data that is not being tracked by the context has the necessary action applied. If you use EF Core to create the database, this cascade behavior will be setup for you.

> [!TIP]  
> You can view this article's [sample](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Saving/Saving/CascadeDelete/) on GitHub.

## Cascading to tracked entities

When you call *SaveChanges*, the cascade delete rules will be applied to any entities that are being tracked by the context.

Consider a simple *Blog* and *Post* model where the relationship between the two entities is required. By convention. the cascade behavior for this relationship is set to *Cascade*.

The following code loads a Blog and all its related Posts from the database (using the *Include* method). The code then deletes the Blog.

[!code-csharp[Main](../../../samples/core/Saving/Saving/CascadeDelete/Sample.cs#CascadingOnTrackedEntities)]

Because all the Posts are tracked by the context, the cascade behavior is applied to them before saving to the database. EF therefore issues a  *DELETE* statement for each entity.

``` sql
   DELETE FROM [Post]
   WHERE [PostId] = @p0;
   DELETE FROM [Post]
   WHERE [PostId] = @p1;
   DELETE FROM [Blog]
   WHERE [BlogId] = @p2;
```

## Cascading to untracked entities

The following code is almost the same as our previous example, except it does not load the related Posts from the database.

[!code-csharp[Main](../../../samples/core/Saving/Saving/CascadeDelete/Sample.cs#CascadingOnDatabaseEntities)]

Because the Posts are not tracked by the context, a *DELETE* statement is only issued for the *Blog*. This relies on a corresponding cascade behavior being present in the database to ensure data that is not tracked by the context is also deleted. If you use EF to create the database, this cascade behavior will be setup for you.

``` sql
   DELETE FROM [Blog]
   WHERE [BlogId] = @p0;
```
