# map-multi-linestring
![Map Multi Linestring, awesome resource field for Nova](banner.jpg)

---

[![Version](http://poser.pugx.org/wm/map-multi-linestring/version)](https://packagist.org/packages/wm/map-multi-linestring)

- [Requirements](#requirements)
- [Installation](#installation)
- [Develop](#develop)
- [Usage](#usage)
  - [Map Multi Linestring](#map-multi-linestring)
- [Configuration](#configuration)

## Requirements

- `php: ^8`
- `laravel/nova: ^4`

## Installation

You can install the package in to a Laravel app that uses [Nova](https://nova.laravel.com) via composer:

```bash
composer require wm/map-multi-linestring
```
## Develop
create a```nova-components``` folder in the root of the project where you want to develop.
Clone map-multi-linestring inside.
add  in ``` "repositories"``` array  attribute of ```composer.json```  
```php 
        {
            "type": "path",
            "url": "./nova-components/map-multi-linestring"
        }

```

modify  in ``` "requires"``` object  attribute of ```composer.json```  
```php 
    "wm/map-multi-linestring": "*",

```
in the first time

launch
```bash
    cd vendor/laravel/nova && npm install
```
we need modify composer.lock 
launch
```bash
    composer update wm/map-multi-linestring
```

## Usage

### Map Multi Linestring

![image](field.png)

You can display a post gist geography(MultiLineString,4326) area on the map and change it by uploading a new MultiLineString file (.GPX, .KML, .GEOJSON)

```php
    use Wm\MapMultiLinestring\MapMultiLinestring;
    /**
     * Get the fields displayed by the resource.
     *
     * @param  \Laravel\Nova\Http\Requests\NovaRequest  $request
     * @return array
     */
    public function fields(NovaRequest $request)
    {
        return [
            ID::make()->sortable(),
                ...
            MapMultiLinestring::make('geometry')->withMeta([
                'center' => [42, 10],
                'attribution' => '<a href="https://webmapp.it/">Webmapp</a> contributors',
                'tiles' => 'https://api.webmapp.it/tiles/{z}/{x}/{y}.png',
                'minZoom' => 5,
                'maxZoom' => 17,
                'defaultZoom' => 10
            ]),
        ];
    }
```
## Configuration

As of v1.4.0 it's possible to use a `Tab` class instead of an array to represent your tabs.

| Property    | Type                | Default     | Description                                                                                                                                                            |
|-------------|---------------------|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| center        | `array`            | `[0,0]`      | The coordinates used for center the view of an empty map                                                                                              |
| attribution      | `string` | `<a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery (c) <a href="https://www.mapbox.com/">Mapbox</a>`      | the html showed as map attribution                                   |
| tiles  | `string` | `https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png`      | The tile url used.                                    |

