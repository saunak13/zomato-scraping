# zomato-scraping
hear we are try to scrap zomato for Resturants near me
#Required packages
import pandas as pd
import bs4
import selenium
import requests
from selenium import webdriver
import re
import time
import itertools
browser=webdriver.Chrome("chromedriver")
#The chrome driver should be in the same folder as the python script file.
browser1=webdriver.Chrome("chromedriver")
url="https://www.zomato.com/bangalore/great-food-no-bull"
browser.get(url)
#####
main_tag=browser.find_elements_by_css_selector('div.col-s-8')
restaurant=pd.DataFrame(columns=['Id','Name','Url'])
for rest in main_tag:
    rest_id=rest.find_element_by_css_selector('div.relative').get_attribute('data-res-id')
    name=rest.find_element_by_css_selector('div.res_title').get_attribute('innerHTML')
    url=rest.find_element_by_css_selector('a').get_attribute('href')
    temp={'Id':rest_id,'Name':name,'Url':url}
    restaurant=restaurant.append(temp,ignore_index=True)
#####
#for url in restaurant.Url:
usr_reviews=pd.DataFrame(columns=['rest_id','user_id','user_name','rating','review'])
i=0
for url in list(restaurant.Url):
    i=i+1
    print(i)
    browser1.get(url)
    main_tag=browser1.find_elements_by_css_selector('div.notifications-content')
    if len(main_tag)==0:
        link=browser1.find_element_by_css_selector('a.result-title.hover_feedback.zred.bold')
        new_url=link.get_attribute('href')
        link.click()
        browser1.get(new_url)
        main_tag=browser1.find_elements_by_css_selector('div.notifications-content')
    for main in main_tag:
        for x in itertools.repeat(1):
            try:
                 main.find_element_by_css_selector('div.load-more.bold.ttupper.tac.cursor-pointer').click()
            except:
                break
        reviews=main.find_elements_by_css_selector('div.ui.segments.res-review-body.res-review')
        for review in reviews:
            try:
                rid=review.get_attribute('data-res_id')
                uname=review.find_element_by_css_selector('div.header.nowrap.ui.left').text
                uid=review.find_element_by_css_selector('div.zs-follow-btn-container').get_attribute('data-user-id')
                rating=review.find_element_by_css_selector('div.ttupper.fs12px').get_attribute('aria-label')
                review_text=review.find_element_by_css_selector('div.rev-text.mbot0').text
                cur_row={'rest_id':rid,'user_id':uid,'user_name':uname,'rating':rating,
                    'review':review_text}
                if uid not in list(usr_reviews.user_id):
                    usr_reviews=usr_reviews.append(cur_row,ignore_index=True)
            except:
                break
####
 restaurant.to_csv('rest.csv')
usr_reviews.to_csv('usr_rev.csv')
#####
restaurant.to_csv('restaurant.csv')
#####

usr_reviews.groupby('rest_id')['rest_id'].count()
#####


 
