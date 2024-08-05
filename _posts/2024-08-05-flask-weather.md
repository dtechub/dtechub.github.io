---
title: Flask weather dashboard
author: petman
date: 2024-08-05 14:10:00 +0800
categories: [Flask, Web programming]
tags: [web]
render_with_liquid: false
---

This tutorial will guide you how to build a `weather dashboard` with Flask, a Python framework for developing web applications.
A `weather dashboard` is an interface or application designed to display weather-related information in a clear, organized, and user-friendly manner. It typically aggregates and presents various weather metrics such as temperature, humidity, windspeed, etc which help users understand current and forecasted weather conditions. 

## What is Flask
Flask is a lightweight and flexible web framework for Python that is used to build web applications easily. Flask is small, easy-to-extend, and easy to use.

## Environment setup
- Start by creating a folder named `weather-dashboard` which will contain all the project's code. Open a terminal (as admin in Windows) and navigate to this directory with a command like: `cd C:Users\petman\Desktop\weather-dashboard`.
- Install Python on your system if it is absent. See [Python Installation](https://www.python.org/downloads/).
- Install and activate a Python virtual environment in this folder. 

```bash
py -m pip install virtualenv # Linux/macOS: python3 -m pip install virtual
py -m virtualenv myenv # Linux/macOS: virtualenv myenv
.\myenv\Scripts\activate # Linux/macOS: source myenv/bin/activate
```
> Using a virtual environment is good practice in Python programming because it isolates your project-specific dependencies from other system dependencies. This prevents conflicts between different projects that may require different versions of the same package.
{: .prompt-info }


- Install Flask and other dependencies.

```bash
py -m pip install Flask requests # Linux/macOS: pip3 install flask
```
- Sign up for a free API key from a weather service provider like [OpenWeatherMap](https://openweathermap.org/api).
  
## Building the Flask application
- Create a file named `app.py` which contains the main backend logic (e.g., routes, DB information etc) of the web application. Put the following code in `app.py`
  
```python
from flask import Flask, render_template, request
import requests

app = Flask(__name__)

# Replace 'YOUR_API_KEY' with your actual API key
API_KEY = 'YOUR_API_KEY'
BASE_URL = 'http://api.openweathermap.org/data/2.5/weather'

@app.route('/', methods=['GET', 'POST'])
def index():
    weather_data = None
    error = None
    if request.method == 'POST':
        city = request.form.get('city')
        if city:
            response = requests.get(BASE_URL, params={
                'q': city,
                'appid': API_KEY,
                'units': 'metric'
            })
            if response.status_code == 200:
                weather_data = response.json()
            else:
                error = 'City not found or API request failed.'
    
    return render_template('index.html', weather_data=weather_data, error=error)

if __name__ == '__main__':
    app.run(debug=True)

```
- TODO: explain code
- Create a basic HTML template which will be rendered with the weather information.
  
```html
<!-- templates/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Weather Dashboard</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
    <div class="container">
        <h1>Weather Dashboard</h1>
        <form method="POST">
            <input type="text" name="city" placeholder="Enter city name" required>
            <button type="submit">Get Weather</button>
        </form>
        {% if weather_data %}
            <div class="weather-info">
                <h2>Weather in {{ weather_data.name }}</h2>
                <p>Temperature: {{ weather_data.main.temp }} °C</p>
                <p>Weather: {{ weather_data.weather[0].description }}</p>
                <p>Humidity: {{ weather_data.main.humidity }}%</p>
                <p>Wind Speed: {{ weather_data.wind.speed }} m/s</p>
            </div>
        {% elif error %}
            <p class="error">{{ error }}</p>
        {% endif %}
    </div>
</body>
</html>
```
- TODO: add CSS styling

- Run the application with `python app.py` or `py -m flask --app app run`. The web application should be accessible at `http:127.0.0.1:5000`.

