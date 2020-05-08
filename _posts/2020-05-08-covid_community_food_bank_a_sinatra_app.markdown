---
layout: post
title:      "CoVID Community Food Bank: A Sinatra App"
date:       2020-05-08 21:22:56 +0000
permalink:  covid_community_food_bank_a_sinatra_app
---


As I finished up my lessons on using the Sinatra library to make a web application (which I thoroughly enjoyed), I once again was faced with the challenge: **what can I build with these new skills that could be helpful to someone?**

I thought about my first project, the [CoVID-19 Tracking CLI App](https://github.com/jkellyphilly/covid_tracking), that I made to give users a quick and easy way to be presented with summary statistics about COVID. I also thought about what I was doing at the time: 3 times per week, my father and I would meet at a senior center, pick up food, and go on a delivery route to pass out the food to senior citizens in North Philadelphia who couldn't make it out to grocery stores.

On one of these delivery days, a woman asked us if her neighbor could get in touch with the Philly CARES team and have her sign up as well. **At that moment, I realized that it would be very convenient for community members (and volunteers!) to have an application where they could track their delivery requests/meet their community members.** Thus, the CoVID Community Food Bank was born. 

## Building interaction with the database: the models

When considering the models to build for this application, I knew right off the bat that I wanted two different user groups: community members and volunteers. What connects these two groups, I thought? 

#### Simple: *the delivery that the volunteer brings to the member*. 

A member can have many delivery requests (assuming that this CoVID quarantine scenario goes on for a while), and a volunteer can have many deliveries (because I thought about my own experience of passing out food - it's much better to get in your car *once* and knock out a string of deliveries in a row). Also, **a member has many volunteers that they interact with through their deliveries, and a volunteer has many members that they interact with through their deliveries**. 

Amazing! The *has many through* relationship worked because of the way I related my user models (Member and Volunteer) to the Delivery model, but it also passed the real-world litmus test! See a screenshot of my table relations below ([or view the diagram on draw.io](https://drive.google.com/file/d/189nH_AESzexbO6fMeRXuRw8htQbOLH9U/view?usp=sharing))

![](https://imgur.com/a/nCId0Dg)

In order to keep my data clean, I used [ActiveRecord Validations](https://guides.rubyonrails.org/active_record_validations.html) to only save objects to the database if certain criteria were met (namely, that all of the mandatory fields in creating new/editing profiles or delivery requests were filled out). This prevented bad data from persisting to my database, which is a must-have for any web application. 

Additionally, I used [ActiveRecord's has_secure_password macro](https://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html) along with [BCrypt](https://github.com/codahale/bcrypt-ruby) to take users' passwords and store them in a salted, hashed version, thereby preventing handling user's passwords directly or in plain-text form (a very big security risk).

## The logic of the application: the controllers

I used a controller for each model - so I had a `DeliveryController`, `MemberController`, and `VolunteerController`. I followed RESTful naming conventions for the routes I built as best as I could, but there were some exceptions: 
1. A `Delivery` object belongs to both a `Member` and a `Volunteer`. Thus, I wanted to have unique routes for a `member` editing the **delivery details of a delivery** and a `volunteer` editing the **status of the delivery**. So, both the `/deliveries/:delivery_id/edit` (members editing delivery details) and `/deliveries/:delivery_id/volunteer` (volunteers editing status details of the delivery) routes exist! 
2. The forms present in both `/deliveries/:delivery_id/edit` and `/deliveries/:delivery_id/volunteer` submit a PATCH request to `/deliveries/:delivery_id`. This is where I needed to use details stored in the **session hash** in order to determine whether a user is a member or a volunteer. 

Additionally, in the MemberController and VolunteerController classes, I built a helper method designed to check to see if the username already exists in the database (for either the members table or the volunteers table):

```
    def username_already_taken?(username)
      !!Member.find_by(username: username)
    end
```

When implementing this logic for a member or volunteer editing their profiles, I realized that the username of that current user would already be taken (by that user) when trying to edit their own profile and not changing the username, so I had to do an extra to check to allow a user to use the same username **if the current user's username is the "duplicate" username**. 

Quick note: this is what the logic looked like when a member/volunteer signed up or logged in. You can see here that I created a new key called `:user_type` and stored that in the session message to know whether the current user was a member or a volunteer.

```
if @member.save
        session[:user_type] = "member"
        session[:user_id] = @member.id
        session[:message] = "Successfully created member profile - welcome, #{@member.name}!"
        redirect "/deliveries"
```


## Putting it on display: the views

I kept things pretty simple with the layouts that I used, but did have fun implementing my own "status message" feature that appears at various points in the user experience with this application. 

```
    <% if session[:message] %>
      <div class="status_message" style="text-align:center;">
        <br>
        <%= session[:message] %>
        <br><br>
        <% session.delete(:message) %>
      </div>
    <% end %>
```

Essentially, if there was a key in the session hash of `:message`, I display that message on the top of the page, and immediately delete the message key from the hash. Thus, if a user immediately reloads the page after seeing one of these messages, it will disappear, because on the second page load, there does **not** exist a key `:message` in the session hash. 

I am not the most design-inclined developer, so was very thankful to find [Water.css](https://watercss.netlify.app/), a very easy and intuitive external stylesheet that I included in my `layout.erb` file to allow the application to have a clean feel. Check it out!

#### Most importantly, [check out the repo here on Github](https://github.com/jkellyphilly/covid-community) and [check out a short video of how to use the app](https://www.youtube.com/watch?v=y_T4EgFcP7E)! 
