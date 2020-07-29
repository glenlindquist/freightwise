# FreightWise Weather App

Simple app to grab weather data for Brentwood, TN from OpenWeather API.

Displays current temp, feels-like temp, daily high and low, along with icon to indication weather conditions.

## External Libraries

For quick styling/prototyping I pulled in Bootstrap. For quick alerts I pulled in Alertify.js library.

## SMS Feature

To enable SMS, I set up an AWS API gateway endpoint that would trigger an AWS Lambda function that I wrote in Ruby. This function used the Ruby SDK for AWS Pinpoint, which allows you to send one-off SMS messages.