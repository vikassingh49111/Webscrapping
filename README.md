# Webscrapping
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

# encoding=utf8
import sys
reload(sys)
import requests
import time
import codecs
import datetime
from bs4 import BeautifulSoup
import pandas as pd

UTF8Writer = codecs.getwriter('utf8')
default_charset = 'UTF-8'
sys.setdefaultencoding("utf-8")
sys.stdout = UTF8Writer(sys.stdout)


def main():
    cities = [("Toronto", "ON")]

    for category in range(36):
        category = category + 1
        try:
            for (city, state) in cities:
                per_page = 200
                results_we_got = per_page
                offset = 0
                api_key = "a177256417426757e459746329598"

                while (results_we_got == per_page):

                    response = get_results(
                        {"sign": "true", "country": "ca", "city": city, "state": state, "category":category,"radius": 10, "key": api_key,
                         "page": per_page, "offset": offset})
                    time.sleep(1)
                    offset += 1

                    results_we_got = response['meta']['count']
                    for venue in response['results']:
                        group = ""

                        try:

                            if "group"  in venue:
                                address_1 = venue['venue']['address_1']
                            else:
                                address_1 = ['NA']

                        except Exception:
                            pass

                        try:

                            if "group"  in venue:
                                lat = venue['venue']['lat']
                            else:
                                lat = ['NA']

                        except Exception:
                            pass

                        try:

                            if "group"  in venue:
                                lon = venue['venue']['lon']
                            else:
                                lon = ['NA']

                        except Exception:
                            pass

                        try:
                            if "group"  in venue:
                                name = venue['venue']['name']
                        except Exception:
                            pass


                        if "group"  in venue:
                                group = venue['group']['name']

                        date = "error"

                        try:
                            date = datetime.datetime.utcfromtimestamp(venue['time']/1000).strftime('%Y-%m-%d %H:%M:%S')
                        except Exception, e:
                            print 'time not available, ', venue['time']


                        df = pd.DataFrame()

                        df['category'] = [category]
                        df['date'] = [date]
                        df['venue_name'] = [name]
                        df['address'] = [address_1]
                        df['lat'] = [lat]
                        df['lon'] = [lon]
                        df['url'] = [venue['event_url']]
                        df['group_name'] =[BeautifulSoup(group.replace(","," ").lstrip()).get_text().encode('ascii', errors='ignore')]
                        df['event_name']= [BeautifulSoup(venue['name'].replace(","," ").lstrip()).get_text().encode('ascii', errors='ignore')]
                        df['city'] = [city]
                        df['state'] = [state]
                        df['disc'] = [BeautifulSoup(venue['description'].replace("=", "").lstrip()).get_text().encode('ascii', errors='ignore')]

                        with open("F:/Meetupevents.csv", 'a') as f:
                            df.to_csv(f,header=False, index= False)



        except Exception:
            pass

def get_results(params):
    request = requests.get("http://api.meetup.com//2/open_events", params=params)
    data = request.json()
    return data

if __name__ == "__main__":
    main()
