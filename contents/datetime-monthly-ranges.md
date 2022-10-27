---
title: Python Datetime Get Month Ranges
slug: datetime-monthly-ranges

publish_timestamp: March 6, 2017
url: https://www.codingforentrepreneurs.com/blog/datetime-monthly-ranges/

---


Every once and a while you need to get monthly ranges in Python. Below is some code that will help you do exactly that. 

At the end of [Charts.js + Django](https://www.codingforentrepreneurs.com/blog/django-and-chartjs/), I left you with the challenge to try and implement monthly data related to displaying on your charts. This is the Python-specific part... can you use this to integrate it with Django? 


```
# python3

import datetime 

from django.utils import timezone


def get_last_month_data(today):
    '''
    Simple method to get the datetime objects for the 
    start and end of last month. 
    '''
    this_month_start = datetime.datetime(today.year, today.month, 1)
    last_month_end = this_month_start - datetime.timedelta(days=1)
    last_month_start = datetime.datetime(last_month_end.year, last_month_end.month, 1)
    return (last_month_start, last_month_end)


def get_month_data_range(months_ago=1, include_this_month=False):
    '''
    A method that generates a list of dictionaires 
    that describe any given amout of monthly data.
    '''
    today = datetime.datetime.now().today()
    dates_ = []
    if include_this_month:
        # get next month's data with:
        next_month = today.replace(day=28) + datetime.timedelta(days=4)
        # use next month's data to get this month's data breakdown
        start, end = get_last_month_data(next_month)
        dates_.insert(0, {
            "start": start.timestamp(),
            "end": end.timestamp(),
            "start_json": start.isoformat(),
            "end_json": end.isoformat(),
            "timesince": 0,
            "year": start.year,
            "month": str(start.strftime("%B")),
            })
    for x in range(0, months_ago):
        start, end = get_last_month_data(today)
        today = start
        dates_.insert(0, {
            "start": start.timestamp(),
            "start_json": start.isoformat(),
            "end": end.timestamp(),
            "end_json": end.isoformat(),
            "timesince": int((datetime.datetime.now() - end).total_seconds()),
            "year": start.year,
            "month": str(start.strftime("%B"))
        })
    #dates_.reverse()
    return dates_ 


get_month_data_range(months_ago=5, include_this_month=True)


date_data = get_month_data_range(months_ago=24, include_this_month=True)
month_labels = ["%s - %s" %(x['month'], x['year']) for x in date_data]
json_ready_month_starts = [x['start_json'] for x in date_data]
print(month_labels)
print(json_ready_month_starts)
large_date_data = get_month_data_range(months_ago=324, include_this_month=True)
print(large_date_data)
```
