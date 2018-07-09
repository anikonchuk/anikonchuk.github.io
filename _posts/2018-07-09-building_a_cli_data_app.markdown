---
layout: post
title:      "Building a CLI Data App"
date:       2018-07-09 15:43:41 +0000
permalink:  building_a_cli_data_app
---


My first major portfolio project through Flatiron School’s online full stack web development program was to build a CLI Data App. The application needs to provide a command line interface and provide access to data scraped from a web page. The data must to go at least one level deep, generally by showing the user a list of available data and then drilling down on a specific item. Finally, the project should use good object-oriented design patterns. So far in the curriculum, our programming has been focused on labs that had clear objectives and tests to guide us to the solution. It was more than a little intimidating to be faced with an empty terminal and have to create the project from scratch. 

#### Setting up the Project
In order to start the project, I needed to figure out my focus.  I needed to find a website that was reasonable to scrape, which was more difficult than I initially thought. The website needed to have HTML and CSS that made the content accessible to the Nokogiri scraping. The first few websites I looked at had a lot of JavaScript, making the task of scraping a bit too tricky for the scope of this project. I finally came across [RottenTomatoes.com's top TV page](https://www.rottentomatoes.com/top-tv/), which seemed to have clear CSS I could use to scrape. 

My app, Rotten Tomatoes’ Fresh TV, would show users RottenTomatoes.com's current TV shows that are “Certified Fresh”, that is, shows that have at least 75% positive reviews from at least twenty sources, including at least five from Top Critics. Users would have the option to get even more information about each show, including a summary of critical reviews and a synopsis. From this starting vision, I planned my CLI. I wanted to have a clear and easy user interface to make the data as accessible as possible. My initial plan was for the following:
* The user would be greeted and presented a list of Rotten Tomatoes’ current “Certified Fresh” TV shows, along with their TomatoMeter scores.
* The user could then choose from the list of TV shows and be presented more information, like a synopsis, the critic’s consensus, and a link for more information. 
* If the user needed to see the list of TV shows again, they could type ‘list’, or they could type ‘exit’ to leave the program.
* Upon typing exit, the user would find a “goodbye” message.
Once I had this initial user experience planned, I began working on building the app itself.

Setting up a completely new file structure was perhaps the most intimidating part of the project. Fortunately, learn.co provided a video tutorial in which Avi Flombaum walked the viewer through the process using bundler (“CLI Gem Walkthrough). By simply typing `bundle gem rt_fresh_tv`, the bundler gem set up the package of files needed to put together a gem of my own. I could just code my classes in the `./lib` directory, as explained by Avi. I also followed his instructions on adding dependencies for files and gems in my environment file (here, `./lib/rt-fresh-tv.rb`) and in the gemspec file. While I successfully added dependencies for Nokogiri, open-uri, and pry, I feel I could still use more practice in this area. 

#### Planning my Classes
Now that my environment was set up, it was time to actually start programming the interface. Here, I also used Avi’s tip to build an interface with “faked” stubbed out data for now, building the functionality around that data. I knew what I wanted the user to see, so I inserted it as “puts” statements until I could figure out how to get the actual data and connect it to the CLI. From here, I decided I needed a few more classes to separate out functionality and responsibility for information. 
* The CLI class is responsible for providing the user interface and interacting with the user. It lists a collection of shows and then pulls specific information for individual shows.
* The Show class holds all the information about individual shows. When a show is instantiated, it is added to an `@@all` class variable. Individual show objects have attributes including title, url,  
* rating, synopsis, and critic_consensus, representing the data to be scraped from the website.
* The Scraper class is responsible for scraping information from the website and creating a new show object from that information.
I set up a basic file structure for these interactions, with each class in its own file. I set up the individual classes to interact with each other, with the CLI class pulling the data from the Show `@@all` class variable, which was to be filled with new instances of Show created by the `scrape_website` method in the Scraper class. 

#### Scraping the Data and Putting it All Together
My next major challenge was scraping the website itself. Each show was contained in its own div element on the website, so I knew I could get Nokogiri to return a collection of the divs. I iterated through each div and pulled the information about title, TomatoMeter rating, url, and critic’s consensus fairly cleanly. The synopsis had to be scraped from the individual show’s page, so I set up an additional scrape within the iteration for Nokogiri to pull that data as well. By using a binding.pry command within the iteration, I was able to check to make sure I was pulling the correct data from the website. 

After successfully finding the scraped data, all that seemed to be left was to connect all the data together and replace the stubbed out data in my CLI class. However, when I tried to run the program, it returned a `No Method` error. I tracked the error back to my scrape_website method in my Scraper class. By using pry, I was able to go back into my iteration to see that the CSS selectors I used were picking up divs later on in the website that did not have the attributes I was using in scraping, so they failed. A simple adjustment to the CSS selectors narrowed the specificity of the scraping. When I went back to try the CLI again, it worked!

The README file provided the next challenge. I was not sure how to describe how to install the actual app. Fortunately, 1:1 Project Support was able to help me see that I was overthinking the problem. Instead of packaging the CLI app as a full gem and submitting to RubyGems.org, it made more sense to simply provide local installation and use instructions. After fixing up the README, it was time to review the project again and see if it could be refactored for clarity and DRY code. 

![](https://i.imgur.com/5aZjvF7.png?2)

#### Refactoring the App and Reflecting on Progress
Looking through the app again, one of the weaknesses of the app is the loading time. Because the scraper needs to run before the shows are listed, it takes a few seconds for the data to populate and the list of shows to appear. This is because the scrape_website method in the Scraper class not only scraped the Top TV shows page, but also each individual show page listed for the synopsis. I knew this was a good opportunity to refactor the code so that scraping each individual web page happened only if that individual show was called for. I moved scraping the actual show's website to a new method (`scrape_show_page`) in my scraper class, and in my CLI class, I only called on that method when an individual show was selected from the list. This sped up my load time considerably and made for a much smoother user interface. 

Through the process of building this data app, I’ve gained a lot of confidence, not only in programming, but also in tackling a larger project. I’ve become more comfortable with iteration and especially web scraping. I feel this project demonstrates solid object-oriented sensibility and design. I look forward to continuing the learn.co program and moving forward on my path to full stack web development. If you’d like to view my source code for this project, it’s available at https://github.com/anikonchuk/rt_fresh_tv-cli-app. 


