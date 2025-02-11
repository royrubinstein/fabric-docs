---
title: Real-Time Intelligence tutorial part 1- Create resources
description: Learn how to create a KQL database and enable data availability in Microsoft Fabric.
ms.reviewer: tzgitlin
ms.author: yaschust
author: YaelSchuster
ms.topic: tutorial
ms.custom:
  - build-2024
  - ignite-2023
ms.date: 11/19/2024
ms.subservice: rti-core
ms.search.form: Get started
# customer intent: I want to learn how to create a KQL database and enable data availability in Microsoft Fabric.
---
# Real-Time Intelligence tutorial part 1: Create resources

> [!NOTE]
> This tutorial is part of a series. For the previous section, see: [Introduction to the Real-Time Intelligence tutorial](tutorial-introduction.md).

In this part of the tutorial, you set up the environment. Specifically, you create an [eventhouse](eventhouse.md), which automatically creates a child KQL database, and then you enable OneLake availability.

## Create an eventhouse

1. Browse to the workspace in which you want to create your tutorial resources. You must create all resources in the same workspace.
1. Select **+ New item**.
1. In the **Filter by item type** search box, enter **Eventhouse**.
1. Select the Eventhouse item.
1. Enter *Tutorial* as the eventhouse name. A KQL database is created simultaneously with the same name.
1. Select **Create**. When provisioning is complete, the eventhouse **System overview** page is shown.

## Turn on OneLake availability

1. From the **System overview** page, select the KQL database, named *Tutorial*, that you created in the previous step.

    :::image type="content" source="media/tutorial/select-tutorial-database.png" alt-text="Screnshot of the System overview for new eventhouse with Tutorial database selected and highlighted with a red box.":::

1. In the righthand pane, under **OneLake**, toggle the **Availability** to **On**.
1. Uncheck **Apply to existing tables** and select **Turn on**.

    :::image type="content" source="media/tutorial/one-lake-availability.png" alt-text="Screenshot showing how to turn on OneLake availability.":::

## Related content

For more information about tasks performed in this tutorial, see:

* [Create a database](create-database.md)
* [Turn on OneLake availability](one-logical-copy.md#turn-on-onelake-availability)

## Next step

> [!div class="nextstepaction"]
> [Tutorial part 2: Get data in the Real-Time Hub](tutorial-2-get-real-time-events.md)
