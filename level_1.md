
# Data Science Curriculum

In this curriculum, we'll explore how to build a data science project in Python. Our end 
goal is a simple interactive graphic that predicts how much precipitation we'll receive
 on a certain day given other weather conditions (temperature, humidity, etc.) for that
  day. Along the way we'll see how to scrape, model, and visualize data. Let's get started!

# Level 1: Scraping Data

In this level, we'll obtain the data to analyze for our project. This is an often 
overlooked part of data science, but it's crucial because errors in data collection 
can cause problems for further analysis.

Data originate from many sources and take on a variety of forms. For example, imagine 
collecting the 5 most recent posts made by all of your Facebook friends. Each of these 
posts would then be considered as one data point. We can observe different things about
 these data (the number of words in the post, whether the post contains a photo, what 
 time it was posted, etc.), and these observables are called *variables*. Oftentimes we
  want to observe something about a data point given certain information about it -- in
   the Facebook example, we could try using other information about a specific post to 
   predict whether the post also includes a photo.

For this project, we'll analyze data from [Wunderground](http://www.wunderground
.com/), a website that contains information about the weather. Let's aggregate weather 
datasets over the past couple of years to see if we can predict the amount of 
precipitation on a certain day. 

This brings us to our first important lesson: *obtaining the data must be scalable*. 
Let's think about how we can assemble such a dataset: one way is to navigate the site 
ourselves for each day of interest while keeping track of all the variables we 
need. This probably won't work if we want to analyze a lot of data -- it'll take 
forever! Automating this process, though, should significantly cut down on the amount 
of time we'll need to spend on data aggregation. 

We'll collect the data through a process called **web scraping**. A lot of valuable 
information is contained within webpages, so we can programatically find the data we're
 looking for and extract precisely the information we need.

Let's start by looking at a sample webpage:

![Wunderground Screenshot](wunderground.png)

This webpage come from the following link: "http://www.wunderground.com/history/airport/KNYC/2016/1/1/DailyHistory.html". It looks like the information 
here is nicely organized, which is good for us! The website clearly denotes where the 
temperature for the day is located. Let's dig a little deeper by exploring how the 
information is presented on the website.

![Inspect Screenshot](inspect.png)

Let's right-click on the 38$^\circ$, and then press "Inspect". This will allow us to 
look at the actual HTML code that generated the webpage we're looking at. We can then 
right-click where it says tbody and press "Edit as HTML".

![Edit as HTML Screenshot](edit-as-html.png)

This should show you the code of this table, and the first lines should look like this:
```
<tbody>
		<tr>
		<td class="history-table-grey-header">Temperature</td>
		<td colspan="3" class="history-table-grey-header">&nbsp;</td>
		</tr>
		<tr>
		<td class="indent"><span>Mean Temperature</span></td>
		<td>
  <span class="wx-data"><span class="wx-value">38</span><span class="wx-unit">&nbsp;°F</span></span>
</td>
		<td>
  <span class="wx-data"><span class="wx-value">33</span><span class="wx-unit">&nbsp;°F</span></span>
</td>
		<td>&nbsp;</td>
		</tr>
		<tr>
		<td class="indent"><span>Max Temperature</span></td>
		<td>
  <span class="wx-data"><span class="wx-value">42</span><span class="wx-unit">&nbsp;°F</span></span>
</td>
		<td>
  <span class="wx-data"><span class="wx-value">39</span><span class="wx-unit">&nbsp;°F</span></span>
</td>
		<td>
  <span class="wx-data"><span class="wx-value">62</span><span class="wx-unit">&nbsp;°F</span></span>
(1966)</td>
		</tr>
```

Here we're seeing exactly how the information is being presented to the browser. It may
 be a little tough to see at this level, but each row of the table (starts 
 with `<tr>` and ends with `</tr>`) has tell us more about the day that we may be 
 interested in. This means that we can write a program that will extract 
 the data we want. Awesome!

Now let's think about the scope of this project -- we probably want at least a 
couple of years' worth of data for meaningful analyses. Let's say that we're 
interested in data between January 1, 2013 and December 31, 2015. Now, we just need to 
write code that'll allow us to get all the links to the data we want. If you remember 
from before, the link was 
 "http://www.wunderground.com/history/airport/KNYC/2016/1/1/DailyHistory.html" for 
 January 1<sup>st</sup>, 2016. So it seems likely that we only need to substitute the 
 year, month, and day that we want data for, which is super convenient. Let's get into 
 some Python code.


    list_of_links = []
    start_year = 2013
    end_year = 2015
    month_to_num_days = {1  : 31, 2  : 28, 3  : 31, 4  : 30,
                         5  : 31, 6  : 30, 7  : 31, 8  : 31,
                         9  : 30, 10 : 31, 11 : 30, 12 : 31}
    link_format = "http://www.wunderground.com/history/airport/KNYC/%d/%d/%d/DailyHistory.html"
    for year in range(start_year, end_year + 1):
        for month in range(1, 12 + 1):
            num_days = month_to_num_days[month]
            for day in range(1, num_days + 1):
                list_of_links.append(link_format % (year, month, day))
        print("Done with %s.." % year)
    print(len(list_of_links))
    for i in range(5):
        print(list_of_links[i])

    Done with 2013..
    Done with 2014..
    Done with 2015..
    1095
    http://www.wunderground.com/history/airport/KNYC/2013/1/1/DailyHistory.html
    http://www.wunderground.com/history/airport/KNYC/2013/1/2/DailyHistory.html
    http://www.wunderground.com/history/airport/KNYC/2013/1/3/DailyHistory.html
    http://www.wunderground.com/history/airport/KNYC/2013/1/4/DailyHistory.html
    http://www.wunderground.com/history/airport/KNYC/2013/1/5/DailyHistory.html


It looks like our code is working! As a sanity check, 365 * 3 = 1095, which means we 
have the expected number of links. Let's start by downloading all of these webpages to 
our computer so that we can extract the information later on. We'll be using the 
`requests` package to download the webpage.

(Note: this will take a while, and that's okay. Shouldn't be more than 10 or so minutes!)


    import requests
    import os.path
    
    def download_file(link, name):
        if os.path.isfile(name):
            return
        file = open(name, 'w')
        r = requests.get(link)
        file.write(r.text)
        file.close()
    for i, link in enumerate(list_of_links):
        if i % 50 == 0:
            print("Done with %d.." % i)
        download_file(link, "%d.html" % i)

    Done with 0..
    Done with 50..
    Done with 100..
    Done with 150..
    Done with 200..
    Done with 250..
    Done with 300..
    Done with 350..
    Done with 400..
    Done with 450..
    Done with 500..
    Done with 550..
    Done with 600..
    Done with 650..
    Done with 700..
    Done with 750..
    Done with 800..
    Done with 850..
    Done with 900..
    Done with 950..
    Done with 1000..
    Done with 1050..


Great, now we have all the data stored locally on our computer! Analyzing the data 
locally will take much less time than calling it from the webpage.

Let's try exploring one of the HTML pages using Beautiful Soup.


    from bs4 import BeautifulSoup
    
    first = open('0.html', 'r')
    soup = BeautifulSoup(first.read(), "html.parser")
    first.close()

Using Beautiful Soup, we can look for particular things on different pages. For 
example, we can look for the links (`a` tags) on the page.


    all_as = soup.find_all('a')
    print(len(all_trs))
    for i in range(5):
        print('-' * 100)
        print(all_as[-i])
        print('-' * 100)

    69
    ----------------------------------------------------------------------------------------------------
    <a href="https://www.wunderground.com/member/registration">
    <i class="fi-torso sidebar-icon"></i> Sign Up / Sign In
      </a>
    ----------------------------------------------------------------------------------------------------
    ----------------------------------------------------------------------------------------------------
    <a href="/wutv/?cm_ven=wutv_toast">
    <iframe class="underlay" frameborder="no" id="ustream-tdu-player" src="http://www.ustream.tv/embed/21416049" width="240"></iframe>
    </a>
    ----------------------------------------------------------------------------------------------------
    ----------------------------------------------------------------------------------------------------
    <a class="modal-close close">×</a>
    ----------------------------------------------------------------------------------------------------
    ----------------------------------------------------------------------------------------------------
    <a aria-label="Close" class="close-reveal-modal">×</a>
    ----------------------------------------------------------------------------------------------------
    ----------------------------------------------------------------------------------------------------
    <a class="button medium radius" href="/member/registration">Remove Ads</a>
    ----------------------------------------------------------------------------------------------------


For the specific data we're looking for, it's all inside of a `table` with 
`historyTable` as the id. We can also search by id with BeautifulSoup.


    main_table = soup.find(id = 'historyTable')

Now we can look for `tr`'s within this table specifically.


    rows = main_table.find_all('tr')
    print(len(rows))
    for i in range(3):
        print('-' * 100)
        print(rows[i])
        print('-' * 100)

    34
    ----------------------------------------------------------------------------------------------------
    <tr>
    <th> </th>
    <th>Actual</th>
    <th>Average </th>
    <th>Record </th>
    </tr>
    ----------------------------------------------------------------------------------------------------
    ----------------------------------------------------------------------------------------------------
    <tr>
    <td class="history-table-grey-header">Temperature</td>
    <td class="history-table-grey-header" colspan="3"> </td>
    </tr>
    ----------------------------------------------------------------------------------------------------
    ----------------------------------------------------------------------------------------------------
    <tr>
    <td class="indent"><span>Mean Temperature</span></td>
    <td>
    <span class="wx-data"><span class="wx-value">33</span><span class="wx-unit"> °F</span></span>
    </td>
    <td>
    <span class="wx-data"><span class="wx-value">33</span><span class="wx-unit"> °F</span></span>
    </td>
    <td> </td>
    </tr>
    ----------------------------------------------------------------------------------------------------


Once we have an actual row, there's lots of information we can extract. Let's try it 
with one of the rows.

(We're only interested in the first two cells because those are the row name and value 
on that day.)


    row = rows[2]
    for cell in row.find_all('td'):
        print('-' * 100)
        print(cell)
        print('-' * 100)
    row_name = row.find_all('td')[0].text.strip() #Get rid of extra whitespace
    row_value = row.find_all('td')[1].text.strip() #Same idea here
    print("%s: %s" % (row_name, row_value))

    ----------------------------------------------------------------------------------------------------
    <td class="indent"><span>Mean Temperature</span></td>
    ----------------------------------------------------------------------------------------------------
    ----------------------------------------------------------------------------------------------------
    <td>
    <span class="wx-data"><span class="wx-value">33</span><span class="wx-unit"> °F</span></span>
    </td>
    ----------------------------------------------------------------------------------------------------
    ----------------------------------------------------------------------------------------------------
    <td>
    <span class="wx-data"><span class="wx-value">33</span><span class="wx-unit"> °F</span></span>
    </td>
    ----------------------------------------------------------------------------------------------------
    ----------------------------------------------------------------------------------------------------
    <td> </td>
    ----------------------------------------------------------------------------------------------------
    Mean Temperature: 33 °F


Wonderful! Let's write some code that can do this for all of the rows in the table we 
had above. 


    def process_row(row):
        if len(row.find_all('td')) == 4: #Only process the rows with 4 cells 
                                    #Eliminates heading rows, etc.
            row_name = row.find_all('td')[0].text.strip()
            row_value = row.find_all('td')[1].text.strip() 
            print("%s: %s" % (row_name, row_value))
        
    for row in rows:
        process_row(row)

    Mean Temperature: 33 °F
    Max Temperature: 40 °F
    Min Temperature: 26 °F
    Heating Degree Days: 32
    Month to date heating degree days: 32
    Since 1 July heating degree days: 1622
    Cooling Degree Days: 0
    Month to date cooling degree days: 0
    Year to date cooling degree days: 0
    Dew Point: 22 °F
    Average Humidity: 54
    Maximum Humidity: 64
    Minimum Humidity: 44
    Precipitation: 0.00 in
    Month to date precipitation: 0.00
    Year to date precipitation: 0.00
    Snow: 0.00 in
    Month to date snowfall: 0.0
    Since 1 July snowfall: 5.1
    Snow Depth: 0.00 in
    Sea Level Pressure: 29.90 in
    Wind Speed: 7 mph
     (WNW)
    Max Wind Speed: 15 mph
    Max Gust Speed: 26 mph
    Visibility: 10 miles
    Events: 


Whoa! We now have tangible data for January 1<sup>st</sup>. For simplicity, let's
 use the 'Mean Temperature', 'Max Temperature', 'Min Temperature', 'Dew Point', 
 'Average Humidity', 'Maximum Humidity', 'Minimum Humidity', 'Precipitation', 'Wind 
 Speed', 'Max Wind Speed', and 'Max Gust Speed' variables (also known as fields) from 
 here out. We can write a function that can scrape one HTML file given its name.


    fields = ['Mean Temperature', 'Max Temperature', 'Min Temperature',\
              'Dew Point', 'Average Humidity', 'Maximum Humidity',\
              'Minimum Humidity', 'Precipitation', 'Wind Speed',\
              'Max Wind Speed', 'Max Gust Speed']
    
    def scrape_file(name):
        f = open(name, 'r')
        soup = BeautifulSoup(f.read(), "html.parser")
        f.close()
        data_rows = soup.find(id = "historyTable").find_all('tr')
        data = {}
        for data_row in data_rows:
            cells = data_row.find_all('td')
            if len(cells) == 4:
                row_name = cells[0].text.strip()
                if row_name in fields:
                    row_value = cells[1].text.split()[0].strip() #Get rid of units
                    data[row_name] = row_value
        return data
    scrape_file("0.html")




    {'Average Humidity': '54',
     'Dew Point': '22',
     'Max Gust Speed': '26',
     'Max Temperature': '40',
     'Max Wind Speed': '15',
     'Maximum Humidity': '64',
     'Mean Temperature': '33',
     'Min Temperature': '26',
     'Minimum Humidity': '44',
     'Precipitation': '0.00',
     'Wind Speed': '7'}



Woo, we're making progress! Now that we can extract the data we want from any general 
HTML, it isn't too much more work to put together all of the data. We'll be storing all
 of this data in a special type of file called a *Comma Separated Values* (CSV) file. A
  CSV file looks something like this:

```
A,B,C
1,2,3
5,10,15
```

It's essentially the same as a spreadsheet that looks like this:

|  A  |  B  |  C  |
| :-: | :-: | :-: |
|  1  |  2  |  3  |
|  5  | 10  | 15  |

CSV files are one of the most commonly-used data formats by data science tools 
today. Let's get started writing out our CSV file!


    csv_file = open('weather_data.csv', 'w')
    csv_file.write('Month,Day,Year,') #We should also keep track which day each row came from
    csv_file.write(','.join(fields))
    csv_file.write('\n')
    csv_file.close()

The above code writes out the headers for the CSV file. Now we have to write out the 
data for each day we scraped, but this won't be too difficult if we use our `scrape_file` 
function.


    def link_to_date(l):
        parts = l.split('/')
        return (parts[-3], parts[-2], parts[-4]) #(Month, Day, Year)
    
    csv_file = open('weather_data.csv', 'a')
    for i, link in enumerate(list_of_links):
        if i % 50 == 0:
            print('Done with %d..' % i)
        data_for_file = scrape_file('%d.html' % i)
        date_for_file = link_to_date(link)
        csv_file.write('%s,%s,%s,' % (date_for_file[0], date_for_file[1], date_for_file[2]))
        for j, field in enumerate(fields):
            csv_file.write(data_for_file[field])
            if j != len(fields) - 1:
                csv_file.write(',') #Only include a comma if it's not the last field
        csv_file.write('\n')
    csv_file.close()

    Done with 0..
    Done with 50..
    Done with 100..
    Done with 150..
    Done with 200..
    Done with 250..
    Done with 300..
    Done with 350..
    Done with 400..
    Done with 450..
    Done with 500..
    Done with 550..
    Done with 600..
    Done with 650..
    Done with 700..
    Done with 750..
    Done with 800..
    Done with 850..
    Done with 900..
    Done with 950..
    Done with 1000..
    Done with 1050..


The above code will probably take a couple of minutes to run. But once it's done, 
you'll have successfully scraped the Wunderground website for daily weather data from 
2013 to 2015! Feel free to open your CSV file in your spreadsheet program of choice and
 check out your awesome accomplishment.
