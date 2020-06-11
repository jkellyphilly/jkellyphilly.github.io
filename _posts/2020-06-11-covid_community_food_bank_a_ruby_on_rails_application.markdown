---
layout: post
title:      "CoVID Community Food Bank: a Ruby on Rails application"
date:       2020-06-11 23:15:09 +0000
permalink:  covid_community_food_bank_a_ruby_on_rails_application
---


## In troubling times, the power of humanity is through community. And community means showing up for neighbors, friends, or complete strangers. 

At the conclusion of the Sinatra section of the Flatiron curriculum, I found myself in the position of wanting to do "more" to help fight against the deadly coronavirus pandemic, and have been heartbroken to hear statistics and witness firsthand how the pandemic is [disproportionately affecting communities of color](https://www.npr.org/sections/health-shots/2020/05/30/865413079/what-do-coronavirus-racial-disparities-look-like-state-by-state). I decided to build the first version of [CoVID Community Food Bank](https://github.com/jkellyphilly/covid-community) on Sinatra.

At the conclusion of the Rails section in the curriculum, I asked myself the same question that I continually ask myself throughout life: **what can I build or do with the skills I've accrued that could be helpful to someone?**

I have still been partnering with various organizations across North Philadelphia to deliver donated food to those who need it, and the demand has certainly not been decreasing. I knew almost immediately that I wanted to take the same online community idea and build it out in Rails/extend the functionality even more. Thus, the second version of the CoVID Community Food Bank was born. 

As I began working, I quickly noticed a difference in time taken to set up the app between the Sinatra and Rails versions. With Rails, a simple `rails new covid-food-bank` was all it took to generate the "ground layer" of requirements needed to get my app up and running. Previously, I spent the better part of an afternoon setting up all of the environment requirements for my Sinatra application, and here I was just a few minutes in to my Rails project, already having the application up and running!

## Database interaction: models

I decided to keep most of the same database setup that was present in the Sinatra version of this application, with the addition of two new tables in the database: `comments` and `delivery_routes`. 

The inclusion of `comments` was to provide users (both community members receiving the food donations and the volunteers passing them out) a method of communicating with one another that was connected to the application, rather than simply emailing each other. The comments are date- and user-stamped, so users know who made the comment and when it was made. This addition enhances the mission of CoVID Community Food Bank by now including a means of communication within the application itself! 

The inclusion of `delivery_routes` was a simple one: I realized quite quickly that volunteers would get quite overwhelmed by the sheer size of the delivery requests, and it would take manual effort to keep track of which ones were supposed to be delivered on which day. Thus, the `DeliveryRoute` class groups `DeliveryRequest` objects by volunteer and date, so a route `belongs_to` a volunteer and `has_many` delivery requests to be completed on that day. 

Once again, this passed the "does-this-make-sense-in-the-real-world" test - because, due to the setup, *a Volunteer has many delivery requests through a delivery route*, which is true for the real-world volunteering that is happening!

See a screenshot below of my table relations ([or view the diagram on draw.io](https://drive.google.com/file/d/1zm9WFHeR1EnH-rEOgklD9VikC_3FWfrY/view?usp=sharing))


![Imgur](https://i.imgur.com/sF4WluW.png)

In order to prevent bad data from persisting to the database, I used ActiveRecord validations to ensure that certain characteristics of various models were present (i.e. `presence: true`) and to ensure that the username for both community members and volunteers was unique. Furthermore, I created 2 [ActiveRecord custom validations](https://guides.rubyonrails.org/active_record_validations.html#custom-validators) to ensure that 1) a comment could only belong to a community member *OR* a volunteer, but *NOT* both, and 2) a date entered for a `DeliveryRequest` would be in the proper MM/DD/YYYY format *and* had to be a date in the future (it wouldn't make sense to create a request for a delivery for yesterday...). 

For login and authentication, I once again used ActiveRecord's [has_secure_password macro](https://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html) along with [BCrypt](https://github.com/codahale/bcrypt-ruby) to take users' passwords and store them in a salted, hashed version, thereby preventing the handling of users' passwords directly or in plain-text form (a very big security risk). I also gave users the ability to sign up/log in with Github using the [OmniAuth Github gem](https://github.com/omniauth/omniauth-github) and parsing the response to get the user's Github email, name, and "nickname", and assign those to the user's attributes of email, name, and username, respectively. 

**One of the most important things I learned from building this project is the importance of separation of concerns.** The logic for updating statuses of delivery requests, deleting a route and thus updating the delivery requests, and deleting a delivery request was quite a lot to deal with... and took a decent amount of code. Previously, I would have thought to simply write the code where it is needed, *but that is neither extendable for future improvements to the application or easy to de-bug.* As this project has had a good deal more logic than other projects I've completed, it's been crucial for me to keep methods that deal with interacting with data from the database in the `app/models/` directory, and to follow suite when it comes to displaying information to the users (even when complicated - more on that and views/helpers later) and controlling the flow between views. 

## "Flow" of the application: controllers

To repeat myself above: separation of concerns is vital to maintaining clear, understandable, and organized code. Additionally, one important step that I took for making my controller files as clean as possible was to use `private` helper methods and the [Rails before_action](https://guides.rubyonrails.org/action_controller_overview.html) method. The private helper methods helped me use the [strong parameters technique](https://edgeguides.rubyonrails.org/action_controller_overview.html#strong-parameters) in order to ensure that only specified data from the parameters received would be included in the creation or updating of an object. The `before_action` method helped save me from writing the same line of code multiple times; essentially, **before** a block of code in a method is run, the `before_action` method calls another method which runs and *then* the block of code in the original controller action is executed. This was particularly helpful for multiple scenarios in which the parameters would pass the ID of an object to be operated on; I was simply able to define a `get_object` method in each case and use `before_action` to apply to multiple controller methods, like this: 

```
class CommunityMembersController < ApplicationController
  before_action :find_member, only: [:show, :edit, :update]
	
	...
	
	def show
	end
	
	...
	
	private
	
	def find_member
    @community_member = CommunityMember.find(params[:id])
  end
end
```

Another way I developed this application with future collaboration in mind was to follow [RESTful naming conventions](https://restfulapi.net/resource-naming/) for my routes. Additionally, this allowed me to take advantage of nested routing, as shown in the URL of the screenshot below. 

![Imgur](https://i.imgur.com/9SRUc1e.png)

This was accomplished by using the nested routing technique in my `config/routes.rb` file: 
```
  resources :community_members, path: 'community-members' do
     resources :delivery_requests, path: 'delivery-requests', only: [:index, :show, :new]
  end
```

## Presenting to the user: views

I was able to achieve lots of customization for my application through the use of [custom helpers](https://en.wikibooks.org/wiki/Ruby_on_Rails/ActionView/Custom_Helpers) and partial views. The use case is this: I need to show something in a specific way based on certain attributes from the object (or objects) being presented. 

Here's a tangible example: on a `CommunityMember`'s "show" page, I didn't want to show the "Edit profile" link unless the current user *is the CommunityMember that is being viewed*. Certainly, this would be achievable within the `community_member/show.html.erb` file, but that would clutter up a file that has the *specific role to present info to the user* with conditional logic. Instead, I used a helper method that rendered a partial view, thus keeping my "show" file strictly to *presenting information*. Here's a closer look at this example:

```
// app/views/community_members/show.html.erb

...

<%= include_edit_option_member(@community_member) %>

// app/helpers/community_members_helpers.rb

module CommunityMembersHelper

   ...
	 
	 def include_edit_option_member(member)
     if member.is_logged_in(session)
       render partial: "helpers/edit_profile", locals: {type: 'community-members', user_id: member.id}
     end
  end
	
end

// app/views/helpers/_edit_profile.html.erb

<a href="/<%=type%>/<%=user_id%>/edit">Edit your profile</a>
```

Thus, I called the helper method on the current instance of `CommunityMember` (which is the page currently being viewed). This helper method checks to see if that member matches the current user who is viewing the page (that's the #is_logged_in method), and if it is true, renders the link to edit page of the member's profile! All worth it to remove what would be a lot of conditional logic from the view file, and keep the application matching the principle of separation of concerns!

### Thanks for checking out this application - my goal is to continue to build things that make the world a better place. This happens little by little, day by day... until it becomes habit! Collaboration is encourage - check out the repo [here](https://github.com/jkellyphilly/covid-food-bank) on Github and check out a short video of how to use the app [here](https://www.youtube.com/watch?v=JtzQ6qJr1lk)!

Stay healthy!
