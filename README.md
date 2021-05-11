# Building a Stock Visualization Website with Python/Django and Alpha Vantage APIs

![homepage mockup](homepage_layout.png)

Table of content:
- Install dependencies and set up project
- Create database model
- Create frontend UI for the homepage
- Create backend logic
- Set up Django URL routing
- Run the web application locally

## Install Dependencies and set up project
We recommend **Python 3.6 or higher**. If you do not yet have Python installed, please follow the download instructions on the official [python.org](https://www.python.org/downloads/) website. 

Once you have Python installed in your environment, please use your command line interface to install the following 2 Python libraries: 
- [Django](https://www.djangoproject.com/download/): `pip install django`
- [requests](https://pypi.org/project/requests/): `pip install requests`

The `pip` installer above should already be automatically included in your system if you are using Python 3.6 or higher downloaded from python.org. If you are seeing a "pip not found" error message, please refer to the [pip installation guide](https://pip.pypa.io/en/stable/installing/). 

If you haven't done so, please obtain a free Alpha Vantage API key [here](https://www.alphavantage.co/support/#api-key). You will use this API key to query financial market data from the Alpha Vantage APIs as you develop this Python/Django website. 

Now, we are ready to create the Django project! 

Open a new command line window and type in the following prompt: 
```shell
(home) $ django-admin startproject alphaVantage
```

You have just created a blank Django project in a folder called `alphaVantage`. 

Now, let's switch from your home directory to the `alphaVantage` project directory with the following command line prompt: 
```shell
(home) $ cd alphaVantage
```

For the rest of this project, we will be operating inside the `alphaVantage` root directory. 

Now, let's create a `stockVisualizer` app within the blank Django project:  
```shell
(alphaVantage) $ python manage.py startapp stockVisualizer
```

We will also create an HTML file for our homepage. Enter the following 4 command line prompts in order: 

Step 1: create a new folder called "templates"
```shell
(alphaVantage) $ mkdir templates
```

Step 2: go to that folder
```shell
(alphaVantage) $ cd templates
```

Step 3: create an empty `home.html` file in the `templates` folder

If you are in Mac or Linux:  
```shell
(templates) $ touch home.html
```
If you are in Windows:  
```shell
(templates) $ type nul > home.html
```

Step 4: return to our `alphaVantage` root directory
```shell
(templates) $ cd ../
``` 

At this stage, the file structure of your Django project should look similar to the one below. You may want to import the project into an IDE such as PyCharm, Visual Studio Code, or Sublime Text to visualize the file structure more easily. 

```
alphaVantage/
    alphaVantage/
        __init__.py
        asgi.py
        settings.py
        urls.py
        wsgi.py
    stockVisualizer/
        migrations/
            __init__.py
        __init__.py
        admin.py
        apps.py
        models.py
        tests.py
        views.py
    templates/
        home.html
    manage.py
```

Let's take a closer look at some of the key files: 
- `manage.py`: A command-line utility that lets you interact with this Django project in various ways. You can read all the details about manage.py in [django-admin and manage.py](https://docs.djangoproject.com/en/3.2/ref/django-admin/).
- `__init__.py`: An empty file that tells Python that this directory should be considered a Python package. Please keep it empty! 
- `settings.py`: Settings/configuration for this Django project.
- `urls.py`: The URL declarations for this Django project; a “table of contents” of your Django-powered site. 
- `models.py`: this is the file you will use to define database objects and schema
- `views.py`: this is where all the backend logic gets implemented and relayed to the frontend (views)
- `home.html`: the HTML file that determines the look and behavior of the homepage



## Specify the database model
Databases are essential components of most modern web and mobile applications. For our stock visualization website, we will create a simple (two-column) database model to store stock market data. 

Before we create the database model, however, let's open the `settings.py` file and quickly modify the following 3 places in the script: 


1. Near the top of `settings.py`, add a line of code `import os`: 
```python
from pathlib import Path
import os #add this line to settings.py
```

2. Inside the `INSTALLED_APPS`, add the `stockVisualizer` app: 
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'stockVisualizer', #add this line to settings.py
]
```

3. Inside `TEMPLATES`, include the `templates` directory we've created earlier in this project: 
```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')], #modify this line in settings.py
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

Now, let's define a Django database model called `StockData` in `models.py`. 

The model has two fields:
- a `symbol` field that stores the ticker string of the stock
- a `data` field that stores the historical prices and moving average values for a given ticker

```python
#models.py

from django.db import models

# Create your models here.
class StockData(models.Model):
    symbol = models.TextField(null=True)
    data = models.TextField(null=True)
```

With `models.py` updated, let's notify Django about the newly created database model via the following command line prompts: 
```shell
(alphaVantage) $ python manage.py makemigrations
```
```shell
(alphaVantage) $ python manage.py migrate
```

At this stage, your file structure should loook similar to the one below: 
```
alphaVantage/
    alphaVantage/
        __init__.py
        asgi.py
        settings.py
        urls.py
        wsgi.py
    stockVisualizer/
        migrations/
            0001_initial.py #you should see this after running the database migration commands
            __init__.py
        __init__.py
        admin.py
        apps.py
        models.py
        tests.py
        views.py
    templates/
        home.html
    manage.py
    db.sqlite3 #you should see this after running the database migration commands
```

The `db.sqlite3` file indicates that we have registered our `StockData` model in the local SQLite database. As its name suggests, SQLite is a lightweight SQL database frequently used for developing web applications (especially in local test environment). SQLite is automatically included in Django/Python, so there is no need to install it separately :-) 

At this stage, there are only two major steps left:
- Set up the homepage file (home.html) so that we can visualize stock prices and moving averagas interactively
- Create the backend logic (views.py) so that we have the right stock data to feed into the frontend UI

Let's proceed! 

## Set up the homepage 



![homepage mockup](homepage_layout.png)

Open the (empty) `home.html` and paste the following content into it: 
```html
<!DOCTYPE html>
<html>
<head>
    <!--<link rel="stylesheet" href="style.css">-->
    <script src="https://cdn.jsdelivr.net/npm/chart.js@3.2.1/dist/chart.min.js"></script>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <title>Stock Visualizer</title>
</head>

<body>
<h2>Interative Stock Visualizer</h2>
<br>

<label for="ticker-input">Enter Symbol:</label>
<input type="text" id="ticker-input">
<input type="button" value="submit" id="submit-btn">
<br>

<div id="graph-area" style="height:80%; width:80%;">
<canvas id="myChart"></canvas>
</div>
<br>

<div>
    Friendly reminder: if the graphing function stops working after several successful instances, don't worry! It is likely that you have reached the 5 requests/minute rate limit of the free Alpha Vantage API key. The graph should work again in the next minute or after you obtain a <a href="https://www.alphavantage.co/premium/" target="_blank">premium API key</a> with a higher rate limit.
</div>

<script>

    $(document).ready(function(){
        // Right after the page is loaded, we get the stock data (default to AAPL) from the Django backend (the 'get_stock_data' function) for plotting
        $.ajax({
              type: "POST",
              url: "/get_stock_data/",
              data: {
                 'ticker': 'AAPL',
              },
              success: function (res, status) {
                var tickerDisplay = res['prices']['Meta Data']['2. Symbol'];
                var graphTitle = tickerDisplay + ' (data for the trailing 500 trading days)'

                var priceSeries = res['prices']['Time Series (Daily)'];
                var daily_adjusted_close = [];
                var dates = [];

                price_data_parse = function(){
                    for (let key in priceSeries) {
                        daily_adjusted_close.push(Number(priceSeries[key]['5. adjusted close']));
                        dates.push(String(key));
                    }

                };
                price_data_parse();

                var smaSeries = res['sma']['Technical Analysis: SMA'];
                var sma_data = [];

                sma_data_parse = function(){
                    for (let key in smaSeries) {
                        sma_data.push(Number(smaSeries[key]['SMA']));
                    }

                };
                sma_data_parse();



                daily_adjusted_close.reverse().slice(500);
                sma_data.reverse().slice(500);
                dates.reverse().slice(500);

                //make a graph
                var ctx = document.getElementById('myChart').getContext('2d');
                var myChart = new Chart(ctx, {
                type: 'line',
                    data: {
                        labels: dates.slice(-500),
                        datasets: [
                            {
                                label: 'Daily Adjusted Close',
                                data: daily_adjusted_close.slice(-500),
                                backgroundColor: [
                                    'green',
                                ],
                                borderColor: [
                                    'green',
                                ],
                                borderWidth: 1
                            },
                            {
                                label: 'Simple Moving Average (SMA)',
                                data: sma_data.slice(-500),
                                backgroundColor: [
                                    'blue',
                                ],
                                borderColor: [
                                    'blue',
                                ],
                                borderWidth: 1
                            },
                        ]
                    },
                    options: {
                        responsive: true,
                        scales: {
                            y: {
                                //beginAtZero: false
                            }
                        },
                        plugins: {
                            legend: {
                            position: 'top',
                            },
                            title: {
                            display: true,
                            text: graphTitle
                            }
                        }
                    }
                });

              }
        });
    });

    $('#submit-btn').click(function() {
        // when the user specifies a new ticker, we call the Django backend (the 'get_stock_data' function) to get the stock data and refresh the graph. 
        var tickerText = $('#ticker-input').val();
        $.ajax({
              type: "POST",
              url: "/get_stock_data/",
              data: {
                 'ticker': tickerText,
              },
              success: function (res, status) {
                var tickerDisplay = res['prices']['Meta Data']['2. Symbol'];
                var graphTitle = tickerDisplay + ' (data for the trailing 500 trading days)'

                var priceSeries = res['prices']['Time Series (Daily)'];
                var daily_adjusted_close = [];
                var dates = [];

                price_data_parse = function(){
                    for (let key in priceSeries) {
                        daily_adjusted_close.push(Number(priceSeries[key]['5. adjusted close']));
                        dates.push(String(key));
                    }

                };
                price_data_parse();

                var smaSeries = res['sma']['Technical Analysis: SMA'];
                var sma_data = [];

                sma_data_parse = function(){
                    for (let key in smaSeries) {
                        sma_data.push(Number(smaSeries[key]['SMA']));
                    }

                };
                sma_data_parse();



                daily_adjusted_close.reverse().slice(500);
                sma_data.reverse().slice(500);
                dates.reverse().slice(500);

                //make a graph
                $('#myChart').remove(); // this is my <canvas> element
                $('#graph-area').append('<canvas id="myChart"><canvas>');
                var ctx = document.getElementById('myChart').getContext('2d');
                var myChart = new Chart(ctx, {
                type: 'line',
                    data: {
                        labels: dates.slice(-500),
                        datasets: [
                            {
                                label: 'Daily Adjusted Close',
                                data: daily_adjusted_close.slice(-500),
                                backgroundColor: [
                                    'green',
                                ],
                                borderColor: [
                                    'green',
                                ],
                                borderWidth: 1
                            },
                            {
                                label: 'Simple Moving Average (SMA)',
                                data: sma_data.slice(-500),
                                backgroundColor: [
                                    'blue',
                                ],
                                borderColor: [
                                    'blue',
                                ],
                                borderWidth: 1
                            },
                        ]
                    },
                    options: {
                        responsive: true,
                        scales: {
                            y: {
                                //beginAtZero: false
                            }
                        },
                        plugins: {
                            legend: {
                            position: 'top',
                            },
                            title: {
                            display: true,
                            text: graphTitle
                            }
                        }
                    }
                });

              }
        });


    });

</script>

</body>
</html>
```



## Set up Django backend
views.py
```python
from django.shortcuts import render
from django.http import HttpResponse
from django.views.decorators.csrf import csrf_exempt
from .models import StockData

import requests
import json


APIKEY = 'my_alphav_api_key' 
#replace 'my_alphav_api_key' with your actual Alpha Vantage API key obtained from https://www.alphavantage.co/support/#api-key


DATABASE_ACCESS = True 
#if False, the app will always query the Alpha Vantage APIs regardless of whether the stock data for a given ticker is already in the local database


# Create your views here.
def home(request):
    return render(request, 'home.html', {})

@csrf_exempt
def get_stock_data(request):
    if request.is_ajax():
        ticker = request.POST.get('ticker', 'null')
        ticker = ticker.upper()

        if DATABASE_ACCESS == True:
            #checking if the database already has data stored for this ticker before querying the Alpha Vantage API
            if StockData.objects.filter(symbol=ticker).exists(): 
                entry = StockData.objects.filter(symbol=ticker)[0]
                return HttpResponse(entry.data, content_type='application/json')

        output_dictionary = {}

        price_series = requests.get(f'https://www.alphavantage.co/query?function=TIME_SERIES_DAILY_ADJUSTED&symbol={ticker}&apikey={APIKEY}&outputsize=full').json()
        sma_series = requests.get(f'https://www.alphavantage.co/query?function=SMA&symbol={ticker}&interval=daily&time_period=10&series_type=close&apikey={APIKEY}').json()

        output_dictionary['prices'] = price_series
        output_dictionary['sma'] = sma_series

        temp = StockData(symbol=ticker, data=json.dumps(output_dictionary))
        temp.save()

        return HttpResponse(json.dumps(output_dictionary), content_type='application/json')

    else:
        message = "Not Ajax"
        return HttpResponse(message)

```

## URL routing
urls.py
```python
from django.contrib import admin
from django.urls import path
import stockVisualizer.views

urlpatterns = [
    path('admin/', admin.site.urls),
    path("", stockVisualizer.views.home),
    path('get_stock_data/', stockVisualizer.views.get_stock_data),
]
```

## Running the website locally
```shell
(alphaVantage) $ python manage.py runserver
```


You should have a fully functional Django web application at http://localhost:8000/

## References




