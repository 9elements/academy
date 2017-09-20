# Rails Best Practices

Work in progress

## Motivation

While Rails has served us well for smaller web projects, the architecture comes to its limits when applications are growing. We identified the following problems while developing large scale Rails applications:

- Helpers grew beyond manageability; no real vertical separability; (Business) Logic in views
- [Too much logic in models ](too-much-logic-in-models.md)
- Controllers tend to do much, way too much. Same applies to rake tasks!
  - [Params Envy](fat-controllers-params-envy.md)
  - [Business logic that should have located in a model or service](fat-controllers-business-logic.md)
- Frontend editing concerns creep into models
- (Over)use of Callbacks, especially in models
- Too big helpers; Too many helpers
- Application Logic spreads over the Rails app

You know the game. When you start developing your web application you might have some logic that won’t fit in the model since it heavily deals with the presentation (HTML & CSS). The classic Rails way is to put this logic in a helper method (that can be directly used within a view). In our experience this is a code smell since these helpers will have a lot of concerns - they also will grow beyond manageability and testability. They will also spread a lot of application logic over a lot of places.

Example code (TODO migrate):
https://gitlab.9elements.com/employour/meinpraktikum/blob/6c58133fabda326a978b6d0bac5203a168a47eee/app/helpers/companies_helper.rb

# Solution

In ausbildung.de we introduced a widget system. Widgets will be passed all the desired data for rendering. They also have instance methods that encapsulate the logic that is needed in the views. This reduces logic in the templates to a bare minimum.


Example code (TODO come up with a better one):
https://gitlab.9elements.com/employour/ausbildung-3/blob/develop/app/widgets/corporation_tab_bar/widget.rb
