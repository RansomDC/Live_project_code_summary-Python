# Python Live Project

## Introduction
Over a period of a week and a half I worked to create a simple, yet functional Django application for collecting information on job postings. The goal of this was two-
fold.
  First, to provide a working application that could store information about job postings, and provide a place to compare various positions and companies based 
  on several criteria. 
  Second, to expand my understanding of the Django framework by doing tasks in the project, including:
* [CRUD functionality](#CRUD-functionality): Providing ways to create database items populated by user input, read and review data already in the database, update information whether userinputted or not, and delete any information that is no longer relevant.
* [Querying an API](#query-API): Using an API to request information from the job searching website Adzuna and then parsing through that data to aquire the data needed for the user. As well as the ability to automatically save important data from those queries.
* [Styling techniques](#CSS-techniques): Creating a user-friendly GUI for ease of use.
* [Web Scraping](#web-scraping): Learning how to responsibly scrabe websites for useful data.
    
 This project had instructor guidance on what types of tasks to complete, but I chose to use it as an exercise in self-reliance. Making sure that I only asked for
 assistance when absolutely necessary. With that I made it through the 10 day project in only 7.5 days, and only asked for help once.
 
 * [CRUD Functionality](#CRUD-functionality)
    Although allowing users to perform CRUD functionality is one of the most basic requirements of many applications. This was a very interesting challenge for someone that had minimal experience with Django. Working out the layout of the framework and how all of the different aspects communicated with  each other was challenging, but rewarding when it all fell into place. In terms of functionality I used Modelforms for certain parts of the program and generated the forms myself for other parts. As with learning any new language or framework it was a bit of a puzzle, but that is the kind of puzzle I like to solve, so I thought it was quite enjoyable.

 * [Querying an API](#query-API)
    This was my first time making a request to an API for information, and although I had read about API's before I now have a much more thorough understanding of their use. I used the API for the [Adzuna](https://developer.adzuna.com/) job search website. This is a fairly simple API with very clear and helpful documentation. I was able to use it for allow the users of my application to do a simple search for jobs with given keywords and general locations. As shown below I took the data from the search which sent via a POST request and formatted the data so that it would successfully
    work in the request url. When the API response came back I saved the relevant data to a temporary table in the database. Once the results page was loaded those data would be pulled and populate the page each filling a form so that they could be individually saved. Here is the data for saving the API results to the temporary database table.

        description = request.POST['what']
        formattedDescription = (description.replace(" ", "%20")).replace(",", "%2C")
        location = request.POST['location']
        formattedLocation = (location.replace(" ", "%20")).replace(",", "%2C")
        search = [description, location]

        response = requests.get(
            'https://api.adzuna.com/v1/api/jobs/us/search/1?app_id=41b593cb&app_key=58bb774dace8a185a8cc32fbdff00416&results_per_page=5&what={}&where={}&sort_by=date'.format(
                formattedDescription, formattedLocation))

        if not response.status_code == 200:
            return render(request, 'JobScraping/error.html')

        json_data = response.json()
        results = json_data['results']

        for i in results:
            try:
                jobData = Temp(
                    minimum_pay=i['salary_min'],
                    maximum_pay=i['salary_max'],
                    title=i['title'],
                    company=i['company']['display_name'],
                    job_url=i['redirect_url'],
                    date_added=(i['created'])[0:10],
                )
            except:
                jobData = Temp(
                    minimum_pay='',
                    maximum_pay='',
                    title=i['title'],
                    company=i['company']['display_name'],
                    job_url=i['redirect_url'],
                    date_added=(i['created'])[0:10],
                )
            jobData.save()


 * [Styling techniques](#CSS-techniques)
    There are several aspects of this program that I had to style specifically myself, but the majority of the project was styled with Bootstrap 5.
    An important part of this program was making sure that the data presented was easily adjustable and manageable. This was made much easier with the help of the CSS/JS library [Sortable](https://github.com/SortableJS/Sortable).



 * [Web Scraping](#web-scraping)
    Although I did not make the web scraping a primary focus of this particular project. It was a very useful skill to learn. More importantly in reading up on web scraping documentation, I learned a great deal about how to resonsibly scrape the web without it being a burden on the server that I am accessing. Since the example on this website will not be acvtively used I did not set up a regulatory element to control the amount of times that request would be made. But if preparing this project for a real-world deployment I would certainly include a method to preven unnecessary repetetive calls (especially since the source for the call is on the websites main page).

      def JobScraping_home(request):
        page = requests.get('https://forecast.weather.gov/MapClick.php?lat=45.5118&lon=-122.6756#.YqN-yXbMIuU')
        soup = BeautifulSoup(page.content, 'html.parser')
        today = soup.select('li.forecast-tombstone:first-child')

        current = today[0].find(class_='period-name').get_text()
        description = today[0].find(class_='short-desc').get_text()
        temp = today[0].find(class_='temp-high').get_text()

        data = [current, description, temp]
        context = {'data': data}
        return render(request, 'jobScraping/JobScraping_home.html', context)
        

