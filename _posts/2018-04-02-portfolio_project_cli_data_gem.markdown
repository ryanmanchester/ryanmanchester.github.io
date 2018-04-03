---
layout: post
title:      "Portfolio Project: CLI Data Gem "
date:       2018-04-03 00:04:18 +0000
permalink:  portfolio_project_cli_data_gem
---


For my first portfolio project, I wanted to create a CLI gem that would scrape the music review page of my favorite music publication, [Under the Radar](http://undertheradarmag.com).  I like this magazine for its abilty to write honest (though sometimes scathing) reviews about lessner known musical acts, hence under the radar. To pay tribute to the hard-to-please music critics, I decided to create a gem to display these reviews from your terminal. 

The thing I most enjoyed about this project was the top-down approach to building. Top-down as in getting to imagine the final experience with using the program and seeing how when we start with code we wish we had, it helps to guide us toward writing robust, abstract code. 

After imagining how I wanted my gem to behave and scrubbing some initial code, I started to think about a scraper class and what its functions would be.  I decided to make two scraper class methods: one that would create and save new instances of review objects and assign attributes from the main music page, and another that would take a URL as an argument from the first method and retrive the remaining attributes. 

```
class AlbumReviews::Scraper

  def self.review_index_scrape
    doc = Nokogiri::HTML(open("http://www.undertheradarmag.com/reviews/category/music/"))

    doc.css(".teaser").each do |article|
      @review = AlbumReviews::Reviews.new

        @review.artist = article.css(".headline h3 a").children.text
        @review.album = article.css(".headline h4 i a").children.text
        @review.label = article.css(".headline h5").children.text
        @review.date = article.css(".date").children.text
        @review.url = article.css(".headline a").attribute("href").value

        AlbumReviews::Reviews.all << @review
        self.review_profile_scrape(@review.url)
      end
    end

    def self.review_profile_scrape(review_link)
      doc = Nokogiri::HTML(open(review_link))
      @review.author = doc.css(".more-details .byline").children.text
      @review.rating = doc.css("#rating b").text
      @review.article = doc.css("#main > p:nth-child(11)").text
    end

end
```
We can see how through iterating over a Nokogiri array, instantiating an instance of a review inside an instance method, and calling the profile scraper inside the loop we are able to assign real data to attributes of objects. Another fantastic thing about object orientaion programming is object collaboration.  The scraper class method is creating a new instance of a review class within the scraper class! And since we have real data stored inside real instances of the reivew class, we have access to and use that data to control our program inside the CLI class. 

```
  def start
    AlbumReviews::Scraper.review_index_scrape
  end
```

In order to get the programming moving and have access to any data at all, we first have to start up the scraper. The #start method is in the CLI class and will be called in the #call method to have everything collaborating: 

```
class AlbumReviews::CLI

  def call
    puts "This week's album reviews:"
    start
    list_reviews
    menu
  end
```

Now #list_reviews is going to collaborate with the reivews class and save the .all array inside an instance variable so we can pass the array to other methods (like #menu): 

```
  def list_reviews
    @reviews = AlbumReviews::Reviews.all
    @reviews.each.with_index(1) do |review, i|
      puts "#{i}. #{review.artist} - #{review.album} - #{review.author} (#{review.date})"
    end
  end
```

This is the first level of our CLI gem: the list we are able to interact with and extract further information from. Now, on to #menu, where we interact with the program and read actual reviews. 

Inside #menu, we set up a while loop in order to stay in the program WHILE the input from the user is not "exit."  From there, we pass our reviews instance variable from #list_reviews with an index number into a local variable to extract and display the review article and rating: 
```
def menu
    input = nil
    while input != 'exit'
      puts "Which review would you like to read? Or type 'list' to list all albums or 'exit' to leave."
      input = gets.strip.downcase

      if input.to_i > 0
        selected_review = @reviews[input.to_i - 1]
        puts "#{selected_review.artist} - #{selected_review.album} - #{selected_review.author} (#{selected_review.date})"
        puts "#{selected_review.article}"
        puts "#{selected_review.author.gsub("By","").strip}'s rating: #{selected_review.rating} out of 10"
        puts "-------"
      elsif input == "list"
        list_reviews
      else
        puts "Which album? Type 'list' to see all reviews or 'exit'."
      end
    end
  end
```
It's kind of a long method, but it works well. I wanted a few lines of puts to make displaying all of the data easier to read. The result looks something like this: 

```
Which review would you like to read? Or type 'list' to list all albums or 'exit' to leave.
1. Hinds - I Don’t Run - By Stephen Mayne (Apr 02, 2018)
2. Eels - The Deconstruction - By Matt the Raven (Mar 30, 2018)
3. The Voidz - Virtue - By Ian Rushbury (Mar 29, 2018)
4. Frankie Cosmos - Vessel - By Charles Steinberg  (Mar 28, 2018)
5. Sunflower Bean - Twentytwo in Blue - By Michael Watkins (Mar 28, 2018)
6. Guided by Voices - Space Gun - By Ian Rushbury (Mar 27, 2018)
7. Haley Heynderickx - I Need to Start a Garden - By Michael James Hall (Mar 26, 2018)
8. Preoccupations - New Material - By Stephen Mayne (Mar 26, 2018)
9. Yamantaka // Sonic Titan - Dirt - By Austin Trunick (Mar 23, 2018)
10. Jack White - Boarding House Reach - By Conrad Duncan (Mar 22, 2018)

5
Sunflower Bean - Twentytwo in Blue - By Michael Watkins (Mar 28, 2018)
Nonetheless, that record—Human Ceremony—did enough to impress a fair smattering of listeners, to the point where the arrival of their sophomore release feels like an event in a way that so many follow-up releases simply do not.
Michael Watkins's rating: 7 out of 10
-------
```







