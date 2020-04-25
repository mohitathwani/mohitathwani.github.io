---
title: 'Google Sheets as a Database: Introducing the Aspire Budgeting app'
date: 2020-04-25 10:00:00
featured_image: '/images/aspire_blog_post.png'
categories: [Open Source, Swift]
---

Yeah you heard that right! Who in their right mind would think of using Google Sheets as a database for their app?

Well, I did and here’s why. I’ve been a Mint.com user for a really long time but it really wasn’t helping me get on top of my budget and eliminate the paycheck to paycheck cycle.  I googled around and found YNAB which literally means *You Need A Budget* which I indeed did, **but** it’s a paid service costing almost $80 a year. They had a free trial and I was willing to give it a shot. 

When I signed up, it had one fundamental problem that Mint.com had as well and that is syncing with all my banks. If one bank synced well today it wouldn’t sync tomorrow and so the information on my dashboard was always stale and I did not want to pay $80 a year to view stale information. There was the option to manually enter data but again I have to pay for that?

Going back to Google and searching for  “ynab google sheet” I came across this [reddit post](https://www.reddit.com/r/ynab/comments/8jhn3s/simple_google_sheets_ynab_alternative/?utm_source=share&utm_medium=ios_app&utm_name=iossmf).

[Aspire Budget](https://aspirebudget.com/), this is exactly what I was looking for. It’s free, it harnesses the power of Google Sheets, my data is *relatively* safe given that Google has a strong privacy policy **and** I dont have to pay!

After a month of use, during which I’d spend a lot of time in the mornings reconciling my accounts because the Google Sheets app didn’t provide the best mobile interface to enter transactions as they happened. So I decided to do something about it and reached out to [u/Sapphire_Rapids](https://www.reddit.com/u/Sapphire_Rapids/?utm_source=share&utm_medium=ios_app&utm_name=iossmf), the creator of the Aspire Budget sheet for his permission to use the name and logo of the sheet for a free and open source iOS and Android app that could connect to the sheet, allow users to read and write data on the fly from the comfort of a native app with a user interface specifically designed for the Aspire sheet. 

Before diving in head first, I thought it would be a nice idea to come up with a plan for the apps. I talk about them below:


# Project Goals
The goal of this project is simple. It is to foster a positive learning environment for myself and others in the field of app development. 

The project should use the latest APIs available and have a clean, simple and a smooth user experience. 

The project will be split up into 4 phases. 

## Phase 1

1. Ability to connect to Google Drive via The Google iOS SDK. 
2. Ability to read data points of interest from the Dashboard tab of the Aspire Sheet. 
3. Siri Integration. For example, “Hey Siri, how much can I spend on groceries?”
4. Integration with iOS widgets. 
5. Fastlane integration for beta deployments, CI/CD, screenshot creation and beta tester sign up sheet. 
6. No data will be cached in Phase 1. The goal of the next few phases will be to build a solid privacy guideline and strategy. 

## Phase 2

1. Ability to add Transactions. 
2. Ability to perform Category Transfers.
3. Building a privacy guideline and strategy with the community enhance user experience using caching. 

## Phase 3

1. Data synchronization. 
2. Efficient strategy for merging data and conflict resolution between data on the cloud and on the device. 

## Phase 4

1. A complete on boarding flow right from copying the sheet from the Aspire website to the users account and entering all the required data in the Configurations tab. 

Phase 2(3) and Phase 3(1) and Phase 3(2) are too high level at this point and will require some research and expertise.


# Current State of Affairs

The goals mentioned above give me some direction and the app is already available on the App Store and in beta on the Play Store with the ability to:

1. Sign in with Google
2. Link your Aspire Sheet to the app
3. Display your Dashboard information.
4. Add transactions.
5. View account balances.


# Try the apps

If you are interested and would want to take the apps for a spin, you can get it from the [App Store](https://apps.apple.com/us/app/aspire-budgeting/id1486036521), [Play Store](https://play.google.com/store/apps/details?id=com.aspirebudgetingmobile.aspirebudgeting), or if you want to take some risks, join the beta list on [TestFlight](https://testflight.apple.com/join/6Kpe4ue2)

# Contributions

If you would like to help out, check it out on [Github](https://github.com/aspirebudgetingmobile/). There are some great contributors and I would love to have you join in too!


