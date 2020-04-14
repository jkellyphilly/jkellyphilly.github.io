---
layout: post
title:      "Building Something (Somewhat) Useful: My First App"
date:       2020-04-14 14:52:37 -0400
permalink:  building_something_somewhat_useful_my_first_app
---


It's crazy to think that just about a month ago, I started the journey as a Flatiron student. Things didn't take off super quickly right away, but as I learned to manage Learn.co and what the cadence of the program looks like, I found myself having to *force* myself to get up, stretch my legs, and take a break. It is such an amazing feeling to *enjoy* the work that I am doing, and to fully immerse myself in it. 

In making my first app that is truly my own, I wanted to do something that truly felt *useful* - and, it just so happens that I am building this app (and probably all of my portfolio projects for Flatiron) during the coronavirus pandemic. 

I thought to myself: what is one challenge that users who want to know more about COVID-19 consistently face? **I realized that there are so many numbers that get tossed around, and until Google's project there didn't seem to be a user-friendly application that would let a user get just a summary of data by country. I then decided to build my own "COVID-19 Summary" application to let a user quickly and easily get a summary of the global spread or specific statistics by country.**

I knew that since the COVID-19 response and even the understanding of the virus itself was rapidly changing, it would be hard to count on scraping data consistently from a web page. Thus, I opted in to finding an API that would provide me with coronavirus data that could be presented as a summary of the virus to the user. During my first Google search, I happened upon [this COVID19 API on Postman](https://documenter.getpostman.com/view/10808728/SzS8rjbc?version=latest#00030720-fae3-4c72-8aea-ad01ba17adf8), and after reviewing the documentation for how to perform the request, I realized that the details in the data returned were perfect for what I was trying to achieve. 

The next step was setting up the flow for my application. I realized that the global "summary" and each country's "summary" would be very similar to each other - in fact, right off the bat, I realized that I would want getter/setter methods for each of the summary data pieces (name, total confirmed cases, new confirmed cases, etc.)... **but the global and country summaries would both have almost all of these attributes.** Thus, I realized that I could make the CountrySummary class a child of the Summary class so that I could keep my code DRY and avoid repetition!

Here's a diagram of the basic user flow of my program ([view interactive document here](https://drive.google.com/file/d/1DXUYb7vqdP-mhRh-UcmCExSHoJFzUzJG/view?usp=sharing)):

![Imgur](https://i.imgur.com/VzP1bYg.png)

Essentially, my code has 4 main classes: the "driver" class that powers user interaction (defined in `cli.rb`), the "dataloader" class that is responsible for making the API call, parsing through all of the data returned, and creating Summary/CountrySummary objects (defined in `dataloader.rb`), and the Summary and CountrySummary classes, which define the class variables, attributes, and methods required to print all of the information out (defined in `summary.rb` and `countrysummary.rb`, respectively). 

Parsing the API response was tricky, because it turns out that it was returned as a giant string and not as a hash as I had expected. Thus, I spent most of an afternoon and evening slowly yet surely chipping away at parsing the string, and ended up building a few helper methods to avoid repetitive code. Finally, after lots of PRY sessions debugging and trying to figure out exactly how to get the data into a format where I could load it into my Summary and CountrySummary objects effectively, I had my class variables filled with all of the objects I wanted! 

### Although, something didn't look quite right... 

... and after examining the Summary "all" class variable that was supposed to save all of the Summary objects (which should just be the global summary), I noticed that *all of the CountrySummary objects were also gettting stored in that array!*

As I mentioned, I was defining the CountrySummary class as inheriting from the Summary class. Here was what my code was in `summary.rb`: 

```
  def initialize(attributes)
    attributes.each {|key, value| self.send("#{key}=", value)}
    @@all << self
  end
```

But, of course, that couldn't be correct! **I didn't re-define/overwrite the #initialize method in the CountrySummary class, so the CountrySummary instances were getting pushed into the Summary class's "all" class variable.** This is not the behavior I desired - I thought about the problem for a while, and realized that what I really wanted to do in the `#initialize` method was to store the instance being instantiated into the current class's "all" variable. Thus, I changed my code to: 

```
  def initialize(attributes)
    attributes.each {|key, value| self.send("#{key}=", value)}
    self.class.all << self
  end
```

and it worked perfectly! Now, when a CountrySummary object was being instantiated, the code `self.class` would yield the class `CountrySummary`, and thus `self.class.all` yields the CountrySummary "all" class variable. So, my items were being stored correctly! 

### That was a perfect lesson for me in **abstraction**... 
... or, making my code as re-usable as possible in various situations. My first version of the #initialize method was *not* re-usable... I had hard-coded in the storage location as the Summary class variable! Only after making my code *abstract* by defining the "all" variable as "the current class's storage variable" was I able to *re-use* the same method for both of my object types. 

The other methods for printing the information or retrieving a `CountrySummary` object based on its name or country code were relatively straightforward - I'd encourage you to check out [this video](https://www.youtube.com/watch?v=WtBNjnAgj3s) of me walking through the entire code for the project! (Note: that video is only part 1 of 3 - cycle through the YouTube playlist I've made to see all three videos and thus the full walkthrough of my program). 

Once I had everything working, I sat back and took a moment to appreciate this truth: I had just built my own app (mostly) from the ground up (of course, the data acquisition was greatly helped by the documentation on Postman). I'm incredibly proud to have built this tiny app that could actually prove useful for users - as I travel on my own path to programming, I will continually look to build things that have the potential to be useful and impactful. 

Thanks for reading, and stay tuned for more updates on my journey to becoming a software engineer!

** Most importantly, check out the repository [here on Github](https://github.com/jkellyphilly/covid_tracking) or [my explanation of the code on a YouTube video here](https://www.youtube.com/watch?v=WtBNjnAgj3s)! **

