---
date: '2026-02-23T10:27:41+01:00'
draft: false
title: 'Devlog API and data requests'
---

# Pet Project API implimentation
During this semester we need to create our own pet project, which is also going to be the product that we will use for our exam.
I have chosen to make a Danish social media platform.
And in this assignment we were given, we needed to implement an API into our project that would fit in.

Since I already have a plan for what API to implement , but i dont have the knowhow or time to do it right now, I have decided to settle on a weather api for now, that can give me the local weather forecast for the coming week.   

I decided to go with Open-Meteo's weather API, since it suits my needs and is easy to set up.

# An interesting twist 
This is the point in the project where a lot of interesting things start to happen as a new programmer.


After I implemented the base 7 day forecast, I sat down and took a look at the product I actually made, and started thinking about if a user or another programmer actually wanted to use this software.

And I came to the conclusion. No.

there is a critical problem in this software for an everyday user, the program i have made, takes 2 parameters, latitude and longitude,

I thought to myself that I'm never going to be able to convince a normal person to look up the coordinates of a place, every time they want to know the weather, there are just way too many competitors that offer an easier alternative.

So now I have two paths ahead of me.

1. Call it a day, I technically did what the assignment told me to do
2. Implement a fix that makes this a much more viable product

Option one is a very tempting trap, that offers a quick way out and be done for the day, but i think as a good programer, you have to look beyond the minimum requirements for something to be considered done, and instead, focus on making a product you yourself feel proud of

So regardless , I went with option two.

So I started looking for an API that made it possible to enter the name of a Danish city, and it would return the coordinates for the city.

This however led to some other interesting problems down the road that I will cover in a later chapter.

To my luck I found out that Open-Meteo had another API that offers this exact service.

# Deep dive
Now lets take a closer look at what i coded specificly, 

Lets start by taking a closer look at the base components i used.

```java
public class ApiService 
{
static ObjectMapper objectMapper = new ObjectMapper();
    public ApiService() {}

    public static JsonNode getApiData (String url)
    {
        try 
        {
            JsonNode node = objectMapper.readTree(new URI(url).toURL());
            return node;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

Here we see the `ApiService`, this class has only one method and one responsibility. 

The `getApiData` is a method that takes a URL as a parameter, and then sends a request to the API. 
The URL we send to the API, is already built by another class to ensure that we retrieve the specific data we want. 
So this class only sends the HTTP request and deserializes the JSON response into a JsonNode


Next we have the URL builder
```java
public LocationAttributes getLocation(String city, String area) {
        String geoUrl = "https://geocoding-api.open-meteo.com/v1/search?" +
                "name=€" +
                "&count=10" +
                "&language=en" +
                "&format=json";

        String encodedCity = URLEncoder.encode(city, StandardCharsets.UTF_8);
        geoUrl = geoUrl.replace("€", encodedCity);
        JsonNode response = apiService.getApiData(geoUrl);

        GeoData geoData = null;
        try{
            geoData = objectMapper.treeToValue(response, GeoData.class);
        } catch (JsonProcessingException e) {
            throw new RuntimeException(e);
        }       
        LocationAttributes finalLocation = null;
        for (LocationAttributes locationAttributes : geoData.getAttributes()){
            if (locationAttributes.getArea().equalsIgnoreCase(area)){
                finalLocation = locationAttributes;
            }
        }
        return finalLocation;
    }
```
This method `getLocation` belongs to the class `WeatherApi`.
It takes two parameters, a city and an area.

The String `city` is used in the URL.

I made the base URL template `geoUrl`, this is the base URL that will direct us to the API that returns data, but without the query parameter that defines what city we are searching for.

Now it's time to replace the "€" with the String `city` to build a complete URL we can use as an HTTP request.

But this introduces a problem for us Danes, you see, we have a bad habit of giving our cities names that make computers in other countries explode.
This means that we cannot send a request with the query parameter `name=Højby`, since "ø" is not URL-safe

To solve this problem, we use `URLEncoder.encode` to encode the URL-unsafe string into a URL-safe string.

We do this by converting the String into a URL-encoded representation.
This means that we change the representation from a locale-specific format, to a universal language that all computers can read, and thereby the String is now URL-safe 

With this done we can now send a URL-safe HTTP request to the API and get.

we use the previously  mentioned `ApiService` for this 

This gives us a JsonNode that we call `response`, we can use an `objectMapper` to convert this Node into Java objects.

# Object Structure
When we convert from JSON to Java objects, we have to make sure that the data structure is maintained, both before and after the deserialization. 

This structure is maintained by having two DTO classes in our Java program.

A `GeoData` class
``` java
@Data
@JsonIgnoreProperties(ignoreUnknown = true)
public class GeoData {

    @JsonProperty("results")
    public List<LocationAttributes> attributes;
}
``` 

and a `LocationAttributes` class
``` java 
@Data
@JsonIgnoreProperties(ignoreUnknown = true)
public class LocationAttributes {
    @JsonProperty("latitude")
    float latitude;
    @JsonProperty("longitude")
    float longitude;
    @JsonProperty("admin2")
    String Area;
    @JsonProperty("name")
    String city;
    public LocationAttributes() {
    }

    @Override
    public String toString() {
        return "city: " + city +
                "\n" + "Area: " + Area +
                "\n" + "latitude: " + latitude +
                "\n" + "longitude: " + longitude + "\n";
    }
}
```
Two classes are required because of the structure of the JSON returned by the API.  

The response contains a top-level object with a results field, which is a list. Each element in that list represents a single location and is mapped to a LocationAttributes object.

With this setup, when executing:
```java
geoData = objectMapper.treeToValue(response, GeoData.class);
```
We obtain a single `GeoData` object containing a list of all matching results (`LocationAttributes`), where each list entry represents an individual location returned by the API.

But now we arive at our next problem, beause you are propoble asking yourself,
"why would the API return a list when I only want the data for one ciry?".

A lot of cities can share tehe same name, in this instance we get 4 cites named "Højby"

Thus, we get the respons in a list, and sort the data we need later in our backend. 

# Filtering data
Now we have all the data we need, but we still need to filter it. 

This is why `getLocation` has two parameters, one is used to build the URL, the other is used to filter the results.
Every city has an area that is assigned to it, the "Højby" we are looking for, is in the area "Odsherred Kommune".

So we can run this for-each loop to find the specific data we are looking for.
```java
 for (LocationAttributes locationAttributes : geoData.getAttributes()){
            if (locationAttributes.getArea().equalsIgnoreCase(area)){
                finalLocation = locationAttributes;
            }
        }
```
Then finally, we return the city object that matches our conditional statement.

Now we use the data assigned to the `LocationAttributes` object. to call the method 
```java
public WeeklyForcast getWeather(float latitude, float longitude)
```
We use `LocationAttributes.getLatitude()` and `LocationAttributes.getLongitude()`
To fill in the parameters. 

The method `getWeather()` works in the same way as getLocation, but returns a different data type. For this reason, we will not go into further detail here. Anyone interested in the specifics is welcome to review the GitHub repository (see bottom).
When both methods are done. we finally receive the local one-week forecast of the area we specified.


# Conclusion

Looking back, I have learned a lot from this exercise.  
Summarized, the key takeaways are:

- **API Response Handling** – Converting API data from JSON to POJOs efficiently  
- **Programmer Mentality** – Putting in extra effort goes a long way when building reliable solutions  
- **Multi-API Usage** – Connecting multiple APIs can be very powerful when designing features  
- **URL Encoding** – Ensuring HTTP requests are URL-safe and handle special characters correctly  
- **API Requests** – Delegating HTTP communication to `ApiService` to keep responsibilities separated

This project helped reinforce the importance of clean architecture, clear separation of responsibilities, and careful handling of external data sources.

{{< github repo="Freddyboi123/Vores" showThumbnail=true >}}





