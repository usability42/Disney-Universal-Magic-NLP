# Natural Language Processing of Theme Park Subreddits

### Exploratory Statement
---
Disney has delighted guests at its theme parks for nearly 50 years.  As we approach our 50th anniversary, it is prudent to reflect upon and assess our brand to determine whether to stay the course or to pivot slightly.

### Executive Summary
---
The covid pandemic has limited guest attendance for the past 18 months.  However, within the next year, we expect to be able to resume business at full capacity.  This window offers us an opportunity to analyze where we are and where we are headed.  We estimate that we have about another year to conduct analysis and implement any brand adjustments we might want to consider before the parks resume business at full capacity, at which point we likely will have other priorities.

The goal is to listen to the voice of the customer.  We have pulled data from the disneyparks subreddit and the universalstudios subreddit to determine if our brand remains distinct from universal's and is consistent with what we consider to be our brand, i.e. Magic.  We have conducted exploratory data analysis on the subreddit data as well as quantitive analysis on the length of subreddit titles and posts, after preliminary research suggested that Universal might have lengthier posts.  We were able to collect the data free of charge.  Since both subreddits are unofficial subreddits not moderated by Disney or Universal, we can trust that they are guests' and potential guests' voices.

Guests invest a lot of money to come to Disney.  We recently increased our single visit park ticket and annual pass costs substantially.  To ensure that this doesn't curb park attendance more than we wish, it's critical to focus on why people come to our parks and how we will continue drawing them to our parks after the pandemics.  Universal, notably, did not increase their admission fees nearly as much as we did.

While Disney and Universal compete in the film sector, and the reddit posts include posts on Disney and Universal movies, the present analysis focuses on branding, particularly at the parks.

**1. Sample Details:**
5,093 posts from r/disneyparks
5,084 posts from r/universalstudios

**2. Our Data:**
Every post has a title.  There is also a selftext field, where users can elaborate on their title, post an image, etc, or leave blank.  Many redditers ask their question or make their statement entirely in the title and leave the selftext field blank.  Moderators can also delete selftexts if they violate the terms of use agreement for the reddit.  After deleting cases with missing selftext, the dataset contained only 1,413 posts from disneyparks and 2,498 from universalstudios.  Fortunately, the title provided richer data anyway, so after some preliminary analysis on the smaller dataset, we focused exclusively on title analysis, enabling us to use the full dataframe (n = 10,177).

Data were pulled using 'https://api.pushshift.io/reddit/search/submission'.

The title and selftext fields were stripped of words that would easily identify them as associated with one of the brands.  These words were:
anaheim, animal kingdom, beijing, blizzard beach, bronze pass, city walk, cruise, disney, disneyland, epcot, eurodisney, express pass, express unlimited, gold pass, hollywood studios, incredi-pass, islands of adventure, land, land paris, magic kingdom, main street, mickey, moscow, park hopper, pirate pass, pixie dust pass, platinum pass, power pass, preferred pass, premier pass, seasonal pass, silver pass, singapore, sorcerer pass, studios hollywood, studios orlando, tokyo land, typhoon lagoon, universal, volcano bay, walt, wdw


We also created the following fields:
- word_count_title = word count of the title fields *
- word_count_selftext = word count of the selftext field *
- disney_y = dummy version of column listing the subreddit the post came from
  - 1 = Disney; 0 = universal
- createdutc = the epoch time when the post was made
- cln_lmn_tok_title = the cleaned, lemmatized, tokenized title
- magic_count = returns True if the word "magic" appears in the cleaned, lemmatized, tokenized title (cln_lmn_tok_title)

* Both of these fields are based on the cleaned, lemmatized, tokenized version of the respective field.

A few other fields were created that we didn't use in our final analysis.


**3. Our Goal**
Create a model that predicts whether a post comes from the disneyparks subreddit or the universalstudios subreddit with a high degree of accuracy.  We expect some overlap, since both Disney and Universal run theme parks and movie franchises.  Still, the better the model, the more defined our brand likely is.

**4. Exploratory Data Analysis**
During exploratory data analysis, we discovered that "magic" was not one of the top 10 most common words in disneyparks titles.  Even though we filtered out the term "Magic Kingdom," since this would be an obvious giveaway, the word "magic" on its own was left in the data.  Since magic lies at the heart of our brand, we decided to double check our analysis.

Using a lambda function, we created calculated a boolean column called "magic," that indicated whether the word "magic" appeared in the title or not.  We added up the total number of Trues for a sum of 225.  Then we multiplied each True or False by the disney_y column and summed the results to get the total number of disneyparks subreddit titles containing "magic" (180).  Lastly, we subtracted 225 - 180 to get universalstudios' magic count of 45.

The good news is that despite Harry Potter's magic wands, 80% of the occurances of magic were in disneyparks.  However, the number remains low.  When redditers post about Disney, they're not mentioning the word "magic" often.  This warrants further investigation (see Conclusions and Recommendations below).

**5. Model Performance**
Since exploratory data analysis indicated a possible correlation between selftext length and subreddit classification, we began with a logistic regression model, with word_count_selftext as the independent variable and disney_y as the dependent variable.  The model failed to classify selftexts any better than the null model (null model = 50%).  I tried the same with word_count_title and got the same non-result.  There is no relationship between selftext length or title length and subreddit.

Second, we tried a Random Forest Classifier using Grid Search and CountVectorizer.  The random forest model classified 78% of the titles accurately.  This is a good start, but we believed we could attain a higher score with a different model.  We first ran an Extra Trees classifier instead of a random forest classifier, but the extra trees classifier "improved" the model by a meager .008.  Clearly this is not a significant improvement.

Third, we tried a multinomial Naive Bayes model.  This model proved far more promising.  Reserving 1/3 of the data for the test dataset, the model yielded a training score of .82 and a test score of .84.  It's unusual for the test data to have a higher score than the training data, but it does happen.  In this case, the difference is minimal and not of significant concern.  Any time data are divided, a little variation is inevitable.
The confusion matrix for the Naive Bayes model is:
- 1,434 true positives = correctly classified reddit as disneyparks
- 1,390 true negatives = correctly classified reddit as universalstudios
- 288 false positives = incorrectly classified reddit as disneyparks
- 247 false negatives = incorrectly classified reddit as universalstudios

Fourth, we tried adding a series of Grid Searches to the multinomial Naive Bayes model.  All scores were close.  The "best" Grid Search (#4) produced the highest accuracy:
- 1,458 true positives = correctly classified reddit as disneyparks
- 1,368 true negatives = correctly classified reddit as universalstudios
- 310 false positives = incorrectly classified reddit as disneyparks
- 223 false negatives = incorrectly classified reddit as universalstudios

Although the accuracy was the same (.84) for the Naive Bayes models with or without the Grid Searches, the Naive Bayes model without the Grid Searches predicted true positives better than the Bayesian model with any of the Grid Searches.  In other words, it has slightly higher precision.  The same model also had better specificity.  The differences are small, but together warrant enough reason to declare The Naive Bayes model without the Grid Searches as the best model.

As mentioned previously, we knew we couldn't achieve a score above 90% because of the overlap between Disney and Universal.  However, we were able to achieve almost 84% accuracy, which suggests that our brands are distinct.

**6. Conclusions and Recommendations**
Aside from Frozen, recent Disney movies have not included a lot of magic.  Mulan disguised herself as a boy.  Monsters Inc features monsters working.  Aside from being able to enter children's rooms through their closets, there's no magic involved.  Inside Out, the Good Dinosaur, Finding Nemo, Finding Dory, Zootopia, Coco, and Moana are all coming of age stories.  Even Mary Poppins Returns features magic, but ultimately is a coming of age story.  Aside from toys and cars coming alive, there's no magic in the Toy Story or Cars movies.

Disney recently renamed all of their annual passes (and greatly increased the price on each of them).  Instead of bronze, silver, gold, and platinum passes, there are the Pixie dust, Pirate, Sorcerer, and Incredi-pass.  I'm guessing that the last one is promoting The Incredibles.  Renaming the passes to align with the magic theme (or a new exhibit) signals that they're doubling down on their Magic branding.

If Disney chooses to pivot its brand slightly, then I recommend clarifying what that brand is, based on where they are and what new content is in the works.  One possibility is Imagination.  This would tie in well with EPCOT (which desperately needs clear branding, but that's beyond the scope of this project) and Disney's Imagineering work group, which is considered the heart of Disney design, according to a Disney+ documentary.  "Dream it" or "Dream it, become it" could also work.

At the heart of the question lies guests' and potential guests' view of "magic."  Is it passe?  When guests hear the word "magic," do they think of older films with evil witches and good fairies?  While I doubt evil witches and good fairies will die out completely, they are not highly represented in Disney's newer films.  Perhaps "magical" no longer represents Disney best.  Alternatively, perhaps Disney seeks to redefine or expand the definition of Magic.  In any case, further research is warranted.  I am not privy to proprietary data, but I can make recommendations based on publicly available data and information.

If I were a marketing data science cast member at Disney, I would recommend researching other unofficial Disney subreddits and social media pages to examine the most frequently discussed topics and posted images.  Then I'd perform a meta-analysis on all that information to get an updated view of the Disney landscape in recent past, current, and potential customers' eyes.  What strikes them about Disney?  What draws them to Disney?  What buzzwords do they repeat most that are uniquely Disney?  Then I would compare these words and themes with words and themes on Universal subreddits and other social media pages to ensure that Disney's brand is unique.  Disney already has researchers collecting data from park guests.  They unobtrusively ask people waiting for friends and family outside the restroom.  A good question to ask is, "What does magic mean to you?" Alternatively, "What comes to your mind when you hear the word 'magic'?"  This data could be analyzed using an NLP model rather than old-fashioned qualitative analysis (which is great, but takes a lot more time).

Collecting and analyzing these data would inform Disney branding executives on where Disney is now in terms of its content, rather than its marketing message, and give a proverbial forest-level view of its landscape to inform its path into the future.
