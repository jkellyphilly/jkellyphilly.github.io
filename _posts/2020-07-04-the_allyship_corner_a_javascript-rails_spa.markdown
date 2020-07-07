---
layout: post
title:      "The Allyship Corner: A JavaScript-Rails SPA"
date:       2020-07-04 18:28:26 -0400
permalink:  the_allyship_corner_a_javascript-rails_spa
---


## After self-education, the best thing to do to become a better ally to oppressed minority groups is to *actually attend events*.

As I wrapped up about 4 weeks of studying & learning JavaScript/front-end programming methods and techniques, I once again asked myself the same question that I continually ask myself throughout my life: **what can I build or do with the skills I've accrued that could be helpful to someone?**

The last month has been a challenging one, but it has also brought me some hope. I am hopeful that real change is coming about in our country, and that those who live with privilege (like myself) will seek to elevate the voices of those who are usually not heard... *and then act on what they hear!*

I built The Allyship Corner to offer a tool for finding & creating events for social justice. A user is able to express interest in attending an event, create a new event, or leave comments on other various events. I sought to bring a sense of community to those who want to work together to make the world a better place, and my hope is that The Allyship Corner can be just that. 

## Rails: setting up the backend
The first step in building this application was setting up my backend API through Rails. Thankfully, Rails has a handy way to spin up an API-only application quickly: `rails new allyship-corner-backend --api`. This does three main things: it configures the application to start with a more limited set of middleware, it makes `ActionController` inherit from `ActionController::API` (rather than `ActionController::Base`), and configures the setup of new resources to skip the creation of views, helpers, and assets. 

I thought critically about the database design I wanted to implement for this project, and I knew that I would have two main database tables: `events` and `comments`. As the project progressed, I also adopted `users` as well in order to determine who would be allowed to delete a certain event & comments. Events would have many comments, and the rest of the event "properties" would just be columns in the event table (i.e. the event's name, location, etc.). An event would *belong to* a user (the user who created the event), and a comment would also *belong to* a user. 

For illustration, see a screenshot below of my table relations ([or view the diagram on draw.io](https://drive.google.com/file/d/18r4SRUjjy32S6BiajRFGONx-3_5NtAov/view?usp=sharing))

![Imgur](https://i.imgur.com/Btqf5lY.png)

I followed RESTful conventions for my routes in the API and quickly was able to seed my database, ensure that the relationships were working correctly, and add validations for my models to prevent bad data from persisting to the database. In order to render JSON in an orderly and structured way (and to clean up the code in my controllers), I used the [FAST JSON API gem](https://github.com/Netflix/fast_jsonapi) from Netflix to quickly serialize my Ruby objects that I was rendering. See an example below from my `Comment` class: I wanted to include the content & time of update attributes as well as the attributes of the comment's event and its user. 

```
class CommentSerializer
  include FastJsonapi::ObjectSerializer
  attributes :content, :updated_at, :event, :user
end
```

### That could have been enough for the purposes of this application, but I wanted to add an extra layer of security to The Allyship Corner's database, so I decided to use [JSON Web Tokens](https://jwt.io/). 

At a very high level, when a user signed up or logged in successfully, I used JWT to encode a secret key into a token that was sent back with the response to the client. Then, on the front-end side, I stored that token in the client's local storage, and every subsequent call to the server had a header in its request that included the token, which was decoded by the server, which (if the token was valid) allowed the request to process, which returned the request object from the database. Beautiful! 

```
headers: {
      Authorization: `Bearer <token-goes-here>`
    }
```

Please check out my [blog post on implementing JWT](https://jkellyphilly.github.io/integrating_json_web_tokens_in_to_a_javascript-rails_application) for a full walk-through of how to use this technique. 

## Building the front-end logic
After the DOM content is loaded, front-end logic is essentially just waiting for user actions and then acting on them. Coding in Ruby and Rails has been a fantastic experience for me that I've really enjoyed, but I was pleasantly surprised at how much I enjoyed coding in JavaScript. 

As much as I enjoyed manipulating the DOM and changing button colors or show/hiding containers, my favorite part of building the front-end portion of this application was hands-down building the functions that would actually interact with the server. I generally followed a similar approach: 
1. When I knew I needed to interact with the server in a particular way, I would create a separate function entirely to handle the process. For example, when creating a new event, I defined a separate function `createNewEvent`. 
2. Within that event, I defined an object called `formData`. This object's keys would be the "payload" of what was actually being received in the server's `params`. 
3. I would define an object called `configObj`, which is the object that is actually sent to the server. Here, I specified the method of the HTTP request, the headers to be included in the request, and the `body` of the request (which is the `formData` I previously defined, converted to JSON)
4. Finally, I would perform the request to the server. This meant I needed to know the endpoint to which I was making the request, and would include the `configObj` defined above. 
5. Then, the response would be converted to JSON.
6. Finally, I would interpret the response and perform whatever action I was trying to do there (i.e. create a new event with the response). In my back-end code, I sent back errors if there was problems saving/updating/deleting an object from the database, so if there was a key of `message` in the response, I knew I should just display that and not take further actions. See below for some pseudocode to help illustrate my point.

Note: I also added `catch` statements after the response was received to alert the user if there was a server error. 

![Imgur](https://i.imgur.com/oNf6g3D.png)

## Adding a sprinkle of CSS: Bootstrap to the rescue
I am sure it is no secret by now, but I do not have much patience with trying to design the layout of a web page. Thankfully, I checked out [Bootstrap](https://getbootstrap.com/) to help me painlessly transform my application into something that is neat and orderly, rather than plain HTML on a page. 

![Imgur](https://i.imgur.com/1uMlS8f.png)

### Thanks for checking out this application - my goal is to continue to build things that make the world a better place. This happens little by little, day by day... until it becomes habit! Collaboration is encourage - check out the repo [here](https://github.com/jkellyphilly/allyship-corner) on Github and check out a short video of how to use the app [here](https://www.youtube.com/watch?v=Gcph4zjZFCQ&t=3s)!
