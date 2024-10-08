---
description: 'Version 8: Module 14: Section 6: Code-along'
---

# Using the Fast JSON API Gem

We've seen that it is entirely possible to create our own service class serializers from scratch. This issue is common enough though that there are some popular standardized serializer options available for us to use. One popular option is the [Fast JSON API](https://github.com/Netflix/fast_jsonapi) gem.

## Introduce the Fast JSON API

The Fast JSON API is a JSON serializer for Rails APIs. It provides a way for us to generate _serializer_ classes for each resource object in our API that is involved in customized JSON rendering. We can use these serializer classes to define the specific attributes we want objects to share or not share, along with things like related object attributes.

The result is that in our controller actions, rather than writing a custom render each time, we write out a serializer for each object once and use Fast JSON API to control the way our data is structured.

## Our Initial Configuration

Before we can see the solution Fast JSON API provides, let's start with our three resources for our birdwatching app: birds, locations, and sightings:

```ruby
class Bird < ApplicationRecord
  has_many :sightings
  has_many :locations, through: :sightings
end
```

```ruby
class Location < ApplicationRecord
  has_many :sightings
  has_many :birds, through: :sightings
end
```

```ruby
class Sighting < ApplicationRecord
  belongs_to :bird
  belongs_to :location
end
```

We also have one customized controller action:

```ruby
class SightingsController < ApplicationController
  def show
    sighting = Sighting.find_by(id: params[:id])
    render json: sighting.to_json(:include => {
      :bird => { :only => [:name, :species] },
      :location => { :only => [:latitude, :longitude] }
    }, :except => [:updated_at])
  end
end
```

This produces a specific set of data with some, but not all, related attributes included:

```javascript
{
  "id": 2,
  "bird_id": 2,
  "location_id": 2,
  "created_at": "2020-03-15T16:38:37.225z",
  "bird": {
    "id": 2,
    "name": "Grackle",
    "species": "Quiscalus Quiscula"
  },
  "location": {
    "id": 2,
    "latitude": 30.26715,
    "longitude": -97.74306
  }
}
```

With just three objects and some minor customization, rendering has become complicated. With Fast JSON API, we can extract and separated this work into serializer classes, keeping our controller cleaner.

## Setting Up Fast JSON API

To include Fast JSON API, add `gem 'fast_jsonapi'` to your Rails project's `Gemfile` and run `bundle install`.

Once installed, you will gain access to a new generator, serializer.

## Implementing the Fast JSON API

With the new serializer generator, we can create serializer classes for all three of our models, which will be available to us in any controller actions later:

```bash
rails g serializer bird
rails g serializer location
rails g serializer sighting
```

Running the above generators will create a `serializers` folder within `/app`, and inside, `bird_serializer.rb`, `location_serializer.rb`, and `sighting_serializer.rb` will have been created. With these serializers, we can start to define information about each model and their _related_ models we want to share in our API.

## Updating the Controller Action

To start using the new serializers, we can update our `render json:` statement so that it initializes the newly created `SightingSerializer`, passing in a variable, just as we did when creating our own service class:

```ruby
class SightingsController < ApplicationController
  def show
    sighting = Sighting.find(params[:id])
    render json: SightingSerializer.new(sighting)
  end
end
```

> **NOTE:** Serializers generated by the Fast JSON API gem have two built-in methods called `serializable_hash` and `serialized_json` which return a serialized hash and a JSON string respectively. However, we don't actually need either of these in this example, as `to_json` will still be called on `SightingSerializer.new(sighting)` implicitly. As we will see, once our serializers are configured and initialized, we will not need to do any additional work.

The `SightingSerializer.new(sighting)` statement can be used on _all_ `SightingController` actions we want to serialize, so if we were to add an `index`, for instance, we just pass in the array of all sightings as well:

```ruby
def index
  sightings = Sighting.all
  render json: SightingSerializer.new(sightings)
end
```

But there is a problem still! If we fire up our Rails server and visit `http://localhost:3000/sightings/2`, all we see is the following:

```javascript
{
  "id": 2,
  "type": "sighting"
}
```

The serializer is working, but it behaves a little differently than we're used to.

## Adding Attributes

When rendering JSON directly, controllers will render all attributes available by default. These serializers work the other way around—we must always specify which attributes we _want_ to include. In our example, bird have `name` and `species` attributes and locations have `latitude` and `longitude` attributes, so to include these would update both serializers. For sightings, we could include the `created_at` attribute:

```ruby
class BirdSerializer
  include FastJsonapi::ObjectSerializer
  attributes :name, :species
end
```

```ruby
class LocationSerializer
  include FastJsonapi::ObjectSerializer
  attributes :latitude, :longitude
end
```

```ruby
class SightingSerializer
  include FastJsonapi::ObjectSerializer
  attributes :created_at
end
```

If we go back and check `http://localhost:3000/sightings/2` again, this time we will see that the `created_at` attribute is present:

```javascript
{
  "id": 2,
  "type": "sighting",
  "attributes": {
    "created_at": "2020-03-15T19:52:37.011z"
  }
}
```

We can also use attributes to access related objects, adding them alongside normal object attributes:

```ruby
class SightingSerializer
  include FastJsonapi::ObjectSerializer
  attributes :created_at, :bird, :location
end
```

This results in our rendered JSON includeing an `"attributes"` object with `"created_at"`, `"bird"`, and `"location"`:

```javascript
{
  "id": 2,
  "type": "sighting",
  "attributes": {
    "created_at": "2020-03-15T19:52:37.011z",
    "bird": {
      "id": 2,
      "name": "Grackle",
      "species": "Quiscalus Quiscula",
      "created_at": "2020-03-15T19:52:37.011z",
      "updated_at": "2020-03-15T19:52:37.011z"
    },
    "location": {
      "id": 2,
      "latitude": 30.26715,
      "longitude": -97.74306,
      "created_at": "2020-03-15T19:52:37.011z",
      "updated_at": "2020-03-15T19:52:37.011z"
    }
  }
}
```

However, here we have no control over what attributes are included in the related objects, so we get _all_ the attributes of `"bird"` and `"location"`.

## Adding Relationships

Object relationships can be included in serializers in two steps. The first step is that we include the relationships we want to reflect in our serializers. We can do this in the same way that we include them in the models themselves. A sighting, for instance, belongs to a bird and a location, so we can update the serializer to reflect this:

```ruby
class SightingSerializer
  include FastJsonapi::ObjectSerializer
  attributes :created_at
  belongs_to :bird
  belongs_to :location
end
```

However, when visiting `http://localhost:3000/sightings/2`, Fast JSON API will display a new `"relationships"` object, but will provide only limited information, like the id of the related object:

```javascript
{
  "id": 2,
  "type": "sighting",
  "attributes": {
    "created_at": "2020-03-15T19:52:37.011z",
  },
  "relationships": {
    "bird": {
      "data": {
        "id": 2,
        "type": "bird"
      }
    },
    "location": {
      "data": {
        "id": 2,
        "type": "location"
      }
    }
  }
}
```

Setting these relationships up is necessary for the second step. Now that we have included relationships connecting the `SightingSerializer` to `:bird` and `:location`, to include attributes from these objects, the recommended method is to pass in a second _options_ parameter to the serializer indicating that we want to _include_ those objects:

```ruby
def show
  sighting = Sighting.find_by(id: params[:id])
  options = {
    include: [:bird, :location]
  }
  render json: SightingSerializer.new(sighting, options)
end
```

The result:

```javascript
{
  "data": {
    "id": 2,
    "type": "sighting",
    "attributes": {
      "created_at": "2020-03-15T19:52:37.011z",
    },
    "relationships": {
      "bird": {
        "data": {
          "id": 2,
          "type": "bird"
        }
      },
      "location": {
        "data": {
          "id": 2,
          "type": "location"
        }
      }
    }
  },
  "included": [{
    "id": 2,
    "type": "bird",
    "attributes": {
      "name": "Grackle",
      "species": "Quiscalus Quiscula"
    }
  },
  {
    "id": 2,
    "type": "location",
    "attributes": {
      "latitude": 30.26715,
      "longitude": -97.74306
    }
  }]
}
```

Because we have a `BirdSerializer` and a `LocationSerializer`, when including `:bird` and `:location`, Fast JSON API will automatically serialize their attributes as well.

## Not Quite the Data Structure We Started With

Using Fast JSON API, with the use of relationships and passing a second parameter, we a able to get the same _data_, but in a much different structure. Fast JSON API is meant to be flexible and easy to implement—and it definitely is! In using Fast JSON API though, we lost the ability to design the structure of our JSON data.

## Conclusion

There is a lot more you can do with the Fast JSON API gem, and it is worth reading through their [documentation](https://github.com/Netflix/fast_jsonapi#table-of-contents) to become more familiar with it. It is possible, for instance, to create entirely custom attributes!

We do not get to choose how data gets serialized the way we do when writing our own serializer classes, but we gain a lot of flexibility by using the Fast JSON API. The Fast JSON API gem provides a quick way to generate and customize JSON serializers with minimal configuration. Its conventions also allow it to work well even when dealing with a large number of related objects.

## Resources

* [Fast JSON API \(GitHub\)](https://github.com/Netflix/fast_jsonapi#table-of-contents)

