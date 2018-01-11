## Relationship among a Component, Model, and Template
In ZK, all data components are designed to accept a separate model object that contains data to be rendered, and the component renders the data model upon a template (what you specify inside `<template>`).

![ ](/images/listmodel-template.png)

This design keeps each part in its single responsibility, so that increases their reusability and decouples the data from a component's implementation.
