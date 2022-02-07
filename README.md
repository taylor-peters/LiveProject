# Python Live Project

## Introduction
For the last two weeks of my time at the tech academy, I worked with my peers in a team developing a Django web application as part of larger group project.  In my application, I was tasked with completing both [back end stories](#back-end-stories) and [front end stories](#front-end-stories).  The appication I created allowed users to create and log into profiles, browse API-generated news events, search for player statistics using Beautiful Soup, and save their favorite players for other users to see.  During this two week sprint, I gained valuable experience working with version control software, operating as part of a team, and completing tasks in a timely manner.  [skills](#other-skills-learned) 

Below are descriptions of the stories I worked on, along with code snippets and navigation links. I also have some full code files in this repo for the larger functionalities I implemented.

## CRUD Functionality

### Create, Read, Update, & Delete

![ezgif com-gif-maker](https://user-images.githubusercontent.com/93218689/152829595-3ad0ff01-df2c-4c51-81c5-95ca8b100632.gif)


## Back End Stories
* [Elite Prospects Web Scraping](#elite-prospects-web-scraping)
* [NHL Roster API](#nhl-roster-api--saving-favorite-players)
* [NHL News API](#nhl-news-api)


### Elite Prospects Web Scraping
Using Beautiful Soup, I was tasked with scraping a webpage for data, then displaying it on my application.  To accomplish this, I used Elite Prospects, a hockey-stats website I have used frequently as a player and a coach.  I wanted to make sure that the data was user-driven, so profile creation required favorite team and a favorite player inputs, which was retrieved with with the associated primary key.  This proved challenging as it required a second scrape, contingent on the results of the first.
      
      def IceHockey_scrapeddata(request, pk):
          player_years = []
          player_teams = []
          player_leagues = []
          player_goals = []
          player_assists = []
          test = []

          details = get_object_or_404(Profile, pk=pk)
          det_dict = {'details': details}

          base_url = "https://www.eliteprospects.com/search/player?q="

          fav_name = str(details.favorite_player)
          split_name = fav_name.split()
          print(len(split_name))
         
Because of the way the data was passed to the view, single-named favorite players caused errors, so I checked the size of the array when the name was split.

          if len(split_name) < 2:
              return render(request, 'IceHockey/IceHockey_error.html', det_dict)
          new_name = split_name[0] + '+' + split_name[1]
          url = base_url + new_name
          result = requests.get(url)
          soup = BeautifulSoup(result.text, "html.parser")

          for link in soup.find_all('a'):
              x = str(link.get('href'))
              if split_name[0].lower() and split_name[1].lower() in x:
                  split_text = x.split("/")
                  break

However, two-named players not found in the database returned errors as well.  So, before performing the second scrape, I checked to make sure that the requiste data had been successfully passed into a new array. 

          if split_text:
              player_id = split_text[4]
              player_name = split_text[5]
              soup2 = "https://www.eliteprospects.com/player/" + player_id + "/" + player_name
              new_result = requests.get(soup2)
              soup = BeautifulSoup(new_result.text, "html.parser")

              for seasons in soup.find_all('span', class_="season"):
                  season = seasons.string
                  player_years.append(season)

              for teams in soup.find_all("td", class_="team"):
                  for spans in teams.find_all('span'):
                      for links in spans.find_all('a'):
                          team = links.string
                          player_teams.append(team)
                          break

              for leagues in soup.find_all("td", class_="league"):
                  for links in leagues.find_all('a'):
                      league = links.string
                      player_leagues.append(league)

              for goals in soup.find_all("td", class_="regular g"):
                  goal = goals.string
                  player_goals.append(goal)

              for assists in soup.find_all("td", class_="regular a"):
                  assist = assists.string
                  player_assists.append(assist)

              zipped_list = zip(player_years, player_teams, player_leagues, player_goals, player_assists)
              context = {'zipped_list': zipped_list, 'details': details}
              return render(request, 'IceHockey/IceHockey_scrapeddata.html', context)
          else:
              return render(request, 'IceHockey/IceHockey_error.html', det_dict)
 
 ![ezgif com-gif-maker (1)](https://user-images.githubusercontent.com/93218689/152829653-27f5d679-3914-4531-be4e-19cd553bc82b.gif)


 ### NHL Roster API & Saving Favorite Players
I was tasked with with finding and consuming an API of my choice, then presenting the data on my application.  Like my [web scraping](#web-scraping) task, I wanted the data to be user-driven.  Similar to the solution to the web scraping problem, this required a second API response, contingent on the intial search.  

      def IceHockey_api_page(request, pk):
          user = get_object_or_404(Profile, pk=pk)
          form = TeamForm()
          roster = []
          player_list = []
          number_list = []
          position_list = []
          position_code = []

          response = requests.get('https://statsapi.web.nhl.com/api/v1/teams')
          teams_info = json.loads(response.text)
          team_list = teams_info['teams']
          for team in team_list:
              teamname = team['name']
              teamid = team['id']

To select the correct team from the user response, I simply passed the information from the primary key into an if statement, then selected the associated team data to populate the second API request.

              if teamname == user.favorite_team:
                  urlid = teamid
                  urlname = teamname

          roster_response = requests.get('https://statsapi.web.nhl.com/api/v1/teams' + '/' + str(urlid) + '/roster')
          roster_info = json.loads(roster_response.text)
          roster_list = roster_info['roster']

          for player in roster_list:
              item = player['person']
              player_list.append(item)
              playername = item['fullName']
              roster.append(playername)

              item2 = player['jerseyNumber']
              number_list.append(item2)

              item3 = player['position']
              position_list.append(item3)
              item4 = item3['code']
              position_code.append(item4)

          zipped_list = zip(roster, position_code, number_list)
          context = {'zipped_list': zipped_list, 'urlname': urlname, 'urlid': urlid, 'user': user, 'form': form}
          return render(request, 'IceHockey/IceHockey_api_page.html', context)


![ezgif com-gif-maker (2)](https://user-images.githubusercontent.com/93218689/152829691-21b4fe13-a4c0-4674-be8a-c7ffc7b9149c.gif)

In our ninth story, we were tasked with saving specific data from our API results.  Since the data I was displaying on the page was user-driven, I had to come up with an efficient way to allow users to select the specific data.  I struggled to find an efficient way to pass the API data from the template into the favorite-saving view.  As a solution, I added a simple form to the same for loop that was displaying the API data.  Each time a player's position, number, and name were displayed, a form was created with those same values as inputs.  The 'Add Fav' button would submit the specific form and pass the data onto the favorite-saving stage.


*Jump to: [Front End Stories](#front-end-stories), [Back End Stories](#back-end-stories), [Other Skills](#other-skills-learned), [Page Top](#python-live-project)*


 ### NHL News API
As an additional feature, I wanted to display recent hockey news stories on the home page.  This required consuming a different API.  I found [newsdata.io](https://newsdata.io/), which allowed me to create my own, filtered, news source and reference the response it created.

   
    def IceHockey_home(request):
        news_titles = []
        news_links = []
        news_dates = []

        response = requests.get(
            'https://newsdata.io/api/1/news?apikey=pub_419404de6e248af4abb353fdc5168853dd51&q=ice%20hockey&country=ca,us&language=en')
        news = json.loads(response.text)
        results = news['results']
        for story in results:
            title = story['title']
            link = story['link']
            date = story['pubDate']

            news_links.append(link)
            news_titles.append(title)
            news_dates.append(date)

        zipped_list = zip(news_titles, news_links, news_dates)
        context = {'zipped_list': zipped_list}

        return render(request, 'IceHockey/IceHockey_home.html', context)


*Jump to: [Front End Stories](#front-end-stories), [Back End Stories](#back-end-stories), [Other Skills](#other-skills-learned), [Page Top](#live-project)*

## Other Skills Learned

* Working with a group of developers to identify front and back end bugs to the improve usability of an application
* Learning new efficiencies from other developers by observing their workflow and asking questions  

  
*Jump to: [Front End Stories](#front-end-stories), [Back End Stories](#back-end-stories), [Other Skills](#other-skills-learned), [Page Top](#live-project)*
