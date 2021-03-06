# Busbud Coding Challenge [![Build Status](https://circleci.com/gh/busbud/coding-challenge-backend-c/tree/master.png?circle-token=6e396821f666083bc7af117113bdf3a67523b2fd)](https://circleci.com/gh/busbud/coding-challenge-backend-c)

## Requirements

Design an API endpoint that provides auto-complete suggestions for large cities.
The suggestions should be restricted to cities in the USA and Canada with a population above 5000 people.

- the endpoint is exposed at `/suggestions`
- the partial (or complete) search term is passed as a querystring parameter `q`
- the caller's location can optionally be supplied via querystring parameters `latitude` and `longitude` to help improve relative scores
- the endpoint returns a JSON response with an array of scored suggested matches
    - the suggestions are sorted by descending score
    - each suggestion has a score between 0 and 1 (inclusive) indicating confidence in the suggestion (1 is most confident)
    - each suggestion has a name which can be used to disambiguate between similarly named locations
    - each suggestion has a latitude and longitude
- all functional tests should pass (additional tests may be implemented as necessary).
- the final application should be [deployed to Heroku](https://devcenter.heroku.com/articles/getting-started-with-nodejs).
- feel free to add more features if you like!

#### Sample responses

These responses are meant to provide guidance. The exact values can vary based on the data source and scoring algorithm

**Near match**

    GET /suggestions?q=Londo&latitude=43.70011&longitude=-79.4163

```json
{
  "suggestions": [
    {
      "name": "London, ON, Canada",
      "latitude": "42.98339",
      "longitude": "-81.23304",
      "score": 0.9
    },
    {
      "name": "London, OH, USA",
      "latitude": "39.88645",
      "longitude": "-83.44825",
      "score": 0.5
    },
    {
      "name": "London, KY, USA",
      "latitude": "37.12898",
      "longitude": "-84.08326",
      "score": 0.5
    },
    {
      "name": "Londontowne, MD, USA",
      "latitude": "38.93345",
      "longitude": "-76.54941",
      "score": 0.3
    }
  ]
}
```

**No match**

    GET /suggestions?q=SomeRandomCityInTheMiddleOfNowhere

```json
{
  "suggestions": []
}
```


### Non-functional

- All code should be written in Javascript
- Mitigations to handle high levels of traffic should be implemented
- Challenge is submitted as pull request against this repo ([fork it](https://help.github.com/articles/fork-a-repo/) and [create a pull request](https://help.github.com/articles/creating-a-pull-request-from-a-fork/)).
- Documentation and maintainability is a plus

### References

- Geonames provides city lists Canada and the USA http://download.geonames.org/export/dump/readme.txt
- http://www.nodejs.org/
- http://ejohn.org/blog/node-js-stream-playground/


## Getting Started

Begin by forking this repo and cloning your fork. GitHub has apps for [Mac](http://mac.github.com/) and
[Windows](http://windows.github.com/) that make this easier.

### Setting up a Nodejs environment

Get started by installing [nodejs](http://www.nodejs.org).

For OS X users, use [Homebrew](http://brew.sh) and `brew install nvm`

Once that's done, from the project directory, run

```
nvm use
```

### Setting up the project

In the project directory run

```
npm install
```

### Running the tests

The test suite can be run with

```
npm test
```

### Starting the application

To start a local server run

```
PORT=3456 npm start
```

which should produce output similar to

```
Server running at http://127.0.0.1:3456/suggestions
```

## Implementation
Using Geoname's search API to retrieve results for a given region name or region name prefix
[Geoname search API](http://www.geonames.org/export/geonames-search.html).
The Geoname API allows to specify the prefix via names_startWith query parameter.
The suggestions API implemented will use 'toponymName' field of Geoname API response in its result.
The city will only be included only if toponymName contains region prefix specified by the user.

In absence of user specified latitude and longitude coordinates, scores are determined based on the population,
otherwise the Euclidean distance of the coordinates from the region is used to determine the score.

Geoname API is known to take too long, even while making a GET request from the browser, causing the tests to occasionally timeout.

I am not quite sure what the purpose of the BusBud Token provided is.

Also I have made a small change to a test case which essentially checks if the name matches the REGEX user provided prefix.
The test is using a regex.test to check if that is the case, however the API returns a string not a regex.
I've changed it to string.match instead, which pretty much checks for the same thing.

I've adhered to AirBnB Javascript style guide wherever it made sense to me.

## Deployment
App has been deployed to Heroku http://fathomless-falls-03862.herokuapp.com/suggestions?q=New&latitude=43.70011&longitude=-79.4163

## Mitigation for high traffic
The implementation chose to make use of an existing Geo name search API instead of reinventing the wheel. However the search API has a slow response time.
To alleviate, a cache for a region prefix has been used to reduce the number of calls made to Geo name.

### Alternate implementation by using cities5000 dump and a trie for prefix based lookup
An alternate implementation can instead periodically download Geo name's cities5000.zip dump, load it into a database.
The Web Service can then -
1. Read from a cache maintained for a given prefix
2. Maintain a trie for all prefixes, city names contaning records for the same.
3. In case of a cache miss, retrieve all records for a given prefix from the trie. Update the cache.
4. If the trie does not have the record, read the data from the database, update the trie and cache.
5. Periodically update cities5000 dump by downloading it from Geonames and periodically update the trie and cache.

