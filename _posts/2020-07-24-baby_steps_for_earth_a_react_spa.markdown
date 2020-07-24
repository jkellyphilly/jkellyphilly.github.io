---
layout: post
title:      "Baby Steps for Earth: a React SPA"
date:       2020-07-24 16:04:18 +0000
permalink:  baby_steps_for_earth_a_react_spa
---

## Taking care of the earth feels like a daunting task... but it doesn't have to be! 

It's no secret that the earth and our climate is in peril. It's something I think about quite often and try to act on - so, at the conclusion of multiple weeks learning about the React framework, I decided to use my new skills to address the climate crisis. I built the Baby Steps for Earth application to give individuals the opportunity to break taking care of the earth into little baby steps... and by selecting just 7, a user can put together a plan of one baby step per day for a week to help the environment!

## Rails: setting up the backend

I began building this application by spinning up a Rails API for the backend with `rails new baby-steps-for-earth-backend --api`. I pondered what I wanted my models in the database to be; I knew that one of the models would have to be the goals themselves, but I wasn't sure what my other models would be. I also wanted to work towards giving the users the ability to filter through the goals present in the database, so I came up with the `Tag` model so that users could group similar goals together.

I realized as I was setting up the backend that a goal should be able to have many tags, which was rather obvious. However, a tag couldn't just belong to *one* `Goal`, because then there would be no way to filter through the goals by tags (since each tag would just belong to one `Goal`). Thus, I knew I needed a **many-to-many** relationship, so I set up a JOIN table (`goal_tags`) in order to relate a goal to its many tags, and to take a single tag and find all of the goals it is related to. 

I also wanted users to have the ability to ultimately build a "plan" for how they would implement these baby steps, and thus the `Plan` model was created. I knew that I wanted plans to have exactly 7 goals, so I set up a validation within the `Plan` model: 

```
class Plan < ApplicationRecord

  ...
   
  validates :goals, length: {is: 7}

  ...

end
```

I also set up methods in my backend to standardize and normalize data flowing in to the database, such as "slugifying" my tags (which just means downcasing and replacing spaces with dashes). This helps provide another layer of protection to ensure that bad data does not persist to my database. 

For illustration on my database relations, see the screenshot below ([or view the diagram on draw.io](https://drive.google.com/file/d/1F3nvtS9XhUDzDd5r_tXQ44VmzSMx75tZ/view?usp=sharing))

SCREENSHOT OF TABLE RELATIONS HERE

## Constructing front-end classes
When setting up the front-end for this application, I realized that I had a lot of freedom in determining how to actually build all of the components that would interact with each other. For instance, theoretically, multiple components could live within the same file/within the same class, even. 

However, in order to keep code from being repetitive and to improve readability/bug-fixing efforts, I wanted to separate functionality into the smallest possible groupings as possible; thus, I ended up with many components. 

**But even in this application's first MVP state, I have already seen the benefits of this approach.** I was able to reuse my `Goal` and `GoalList` components for both the "Explore Goals" section of the application *and* the user's profile page, thus keeping my code from being repetitive and only have one source to troubleshoot if errors occur!

### While building the structure for my front-end, I broke apart components into functional and class components based on the criteria of whether they need to maintain state or not. If a component needed to maintain state, then it would be a class component. However, if the component simply inherited actions or data from its parent and then performed functionality based on that, I made that a functional component.

For my project, a similar pattern was followed. There are 3 "sections" of the single-page application: a place for a user to view all the plans that have been created by other users, a place to view goals/create a new goal/add goals to the current user's plan, and a place to view the profile of the current user (and what plans have been added thus far). Each "section" had a container component where I connected to the Redux store (more on that below) and passed down the corresponding information required for display to the child components. Other class components were components such as the one used for filtering goals; in this case, the component itself needed to maintain state to process what the user was inputting in order to make a fetch request with the correct user input. 

An example of a component that did *not* need to maintain state is the `Plan` component: its job was simply to render the plan that was passed down to it. Note that, unlike a class component, this component simply has an argument ("input") of the `props` passed down to it, and returns JSX which renders. 

SCREENSHOT OF PLAN

## Connecting to the Redux store
In order to avoid creating a full hierarchy and passing props down across multiple components, I used [Redux](https://redux.js.org/) in order to store & access data across various components in my application. Additionally, I used [Redux Thunk](https://github.com/reduxjs/redux-thunk) middleware so that I could perform asynchronous requests to my Redux store. This was particularly helpful when I needed to perform fetch requests to my API - with the Thunk middleware in place, the entire process could be completed asynchonously, resulting in a better user experience as a result of not waiting for the successful response from the database. 

This is how I imported/set up the connection to the store and thunk in `index.js`: 

![Imgur](https://i.imgur.com/UiRLbvf.png)

Connecting a component to the store is useful in that the component's props are connected to the Redux store's data and functions. So, for example, for the `GoalsPage` component, I wanted access to the current `goals` from the Redux store as well as the `loadingGoals` variable. I also wanted access to be able to perform the `fetchGoals`, `createGoal`, and `addGoalToMyPlan` dispatch actions from various child components, so I also connected those functions to the props of `GoalsPage` as well. See the screenshot below for how I defined my `mapStateToProps` and `mapDispatchToProps` functions and implemented the connection to the Redux store.

SCREENSHOT OF MAPS HERE

## React Bootstrap to the rescue

Once again, I went with a quick and easy route to transform my application from plain HTML to something that looks rather presentable, so I used [React Bootstrap](https://react-bootstrap.github.io/) to transform my application to something that looks user-friendly (including making my "new goal" form into a modal!).

SCREENSHOT OF APPLICATION HERE (MODAL OPEN)

### Thanks for checking out this application - my goal is to continue to build things that make the world a better place. This happens little by little, day by day... until it becomes habit! Collaboration is encourage - check out the frontend & backend repos [here](https://github.com/jkellyphilly/baby-steps-for-earth) and [here](https://github.com/jkellyphilly/baby-steps-for-earth-backend) on Github and check out a short video of how to use the app [here](https://www.youtube.com/watch?v=Yd6Rc8FYgyg)!
