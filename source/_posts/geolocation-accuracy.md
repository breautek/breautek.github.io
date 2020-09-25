---
title: Understanding Geolocation Accuracy
categories:
    - Programming
tags:
    - geolocation
    - android
    - ios
    - cordova
    - mobile
date: 2020/8/25
twitter:
    shareid: twsrc%5Etfw
    text: Understanding Geolocation Accuracy
    url: https://breautek.com/2020/08/25/geolocation-accuracy/
---

### Knowledge Prerequisites

Before we continue, I assume you have proficient knowledge in general purpose programming languages. Code examples shown in this article are pseudo code using a C-style syntax.

### Geolocation Accuracy

Geolocation accuracy is something you should program for. You should also expect problems and program appropriate user feedback. GPS technologies is awesome, but they are also many variables that will be outside of your control that can cause poor user experiences.

Most phones have the potential of being accurate up to 3 to 5 meters. That is, the received GPS point is likely to be within a 3 to 5 meter radius of the device's actual location. However the phone will never be consistent accurate. It's up to you to analyze GPS events and determine if that GPS update is usable.

Most <span class="tip" title="Application Programming Interface">API</span>'s have configuration options available to request low precision GPS updates or high precision updates; usually at the cost of battery life. If you only require the general location of the device (such as their country, or perhaps city), then low precision should be sufficient. If you need fine point location, such as what street the device is on, then you'll need high precision.

In most API's, high precision is only a request, and not always honoured. There are a number of reasons for this:

1. User's privacy settings

The user may have their location settings set to only provide low accuracy readings, or they may have location services turned off completely. These settings can vary from device to device, but most devices have privacy options for the user.

2. The environment

Sometimes, even if the user has high accuracy modes enabled, the phone still fails to deliver reliable events. One reason could be of their environment. GPS has a hard time communicationg with the satellites when the user is in a large building, especially if the building is concrete. It is completely normal to see accuracy ranges anywhere from 60 meters to 200 meters. Other obstacles can include being surrounded by tall buildings or mountains, and even stormy weather.

The perfect environment to have stable GPS outsides is outside away from buildings, in clear skies weather.

3. The cold boot

Even in ideal environmental conditions, if the phone hasn't used GPS recently, you could experience a "cold boot". This is a lengthy process of where the phone needs to find a fix to three GPS satellites to perform triangulation. This usually takes 10 to 15 minutes and during this time, you generally won't receive GPS updates, or GPS updates may not be accurate.

You may say, "Wow 15 minutes? That's a long time!", and yes, you would be right. It is a long time. That's why most modern devices have <span class="tip" title="Assisted GPS">A-GPS</span> features, to reduce the reliance on the physical GPS satellites orbiting our planet and use other beacons to assist the GPS receiver. These beacons usually includes cell towers, wifi networks, and even other devices using bluetooth. However, some devices the user has fine control on what they want their phone to connect to, which could limit your ability to take advantage of A-GPS features.

### Handling poor accuracy readings

So what can we do? If you answered "not much", you'd be correct! However, a good app would always deal with poor accuracy readings in one way or another. For some apps, this may be simply ignoring the GPS event and wait for the next one. Other apps may count the erroneous events to decide if they should notify the user about a weak GPS signal. This obviously all depends on the app and their use cases.

Most API's will return GPS data such as this one:

```json
{
    "latitude": 45.9468,
    "longitude": -66.6414,
    "accuracy": 2616
}
```

In this example, we have an `accuracy` reading of `2616` meters (Most APIs are in SI units, but you may refer to the documentation of the API you're using). This means that the coordinate  `45.9468, -66.6414` is confident that it's within `2616` meters of the actual location. See the point and circle below:

![](/images/geolocation_accuracy_radius.png)

As you can see, in this GPS position shows I'm in the small city of Fredericton, however because of the `accuracy` reading, I could be pretty much anywhere in Fredericton. This accuracy level may be fine for some applications, however if more precision is required, then this point is not a good point to use. Therefore, this GPS event should be ignored, or counted as an error point.

### More ways to determine accuracy

Using the `accuracy` field isn't the only way to determine the validity of a GPS update.

Here's a brief list of things you can do:

- Check the timestamp

Sometimes, you may receive events with timestamps that are older than your previous GPS update. This was personally witnessed on some Samsung devices.

- Calculate the speed (distance / time) of your GPS points

While doing this will likely lead to "jerky" behaviour, so it might not be acceptable to be presented in the UI. It can be used to determine significant changes, which may hint at faulty points. Utilizing smoothing algorithms or moving averages may also be desirable.

- Using an error count reset

It's reasonable for the GPS signal to encounter a temporary "blip" in accuracy, such as if the user is driving through a tunnel. Considering counting error points over a timeframe. This will help avoid unnecessary error feedback to the user on short blips, and provide error feedback on longer blips when something may actually be wrong.

- Checking for GPS timeouts

A healthy GPS signal will give about 1 update per second. Some devices however, particularly iOS devices utilizes a minimum movement requirement before sending a GPS update event, in effort to increase battery life. This distance filter can however be disabled, if desired.
