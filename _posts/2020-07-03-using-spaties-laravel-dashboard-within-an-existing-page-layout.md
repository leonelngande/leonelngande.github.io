---
layout: post
hidden: false
title: Using Spatie's Laravel Dashboard within an existing Page Layout
author: Leonel Elimpe
tags:
  - Laravel
  - PHP
last_modified_at: 2020-07-03 03:02:21
---
I've recently added Spatie's [Laravel Dashboard](https://github.com/spatie/dashboard.spatie.be) to a backend admin app and didn't realize it is configured as a full webpage already, with an opening `<html>` tag, `<head>`, and `<body>` all setup.

![Preview of Laravel Dashboard as a full web page](/images/uploads/image-20200703141031319.png)

This meant I couldn't just plug it into my existing page layout.

```html
@extends('layouts.app')

@section('content')
<div class="container-fluid">

    <section class="content-header">
        <h1>Dashboard</h1>
    </section>

    {%raw%}{{-- I WANT TO PLUG THE DASHBOARD IN HERE --}}{%endraw%}
    
</div>
@endsection
```

The above template is of my `home.blade.php` file, and I wanted to add the dashboard just below the `<h1>` section. I initially just added the `<x-dashboard>` tag in there as below.

```html
@extends('layouts.app')

@section('content')
<div class="container-fluid">

    <section class="content-header">
        <h1>Dashboard</h1>
    </section>

    <div class="row">
        <div class="col-md-12">{%raw%}
            <x-dashboard>
                <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExamplePieChart::class}}" position="a1:a2" height="140%"/>
                <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExampleDoughnutChart::class}}" position="b1:b2" height="140%"/>
                <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExamplePieChart::class}}" position="c1:c2" height="140%" />
                <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExampleDoughnutChart::class}}" position="d1:d2" height="140%" />
                <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExampleBarChart::class}}" position="a3:b4" />
                <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExampleLineChart::class}}" position="c3:d4" />
                <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExampleLineChart::class}}" position="a5:b6" />
                <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExampleBarChart::class}}" position="c5:d6" />
                <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExampleBarChart::class}}" position="a7:b8" />
                <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExampleLineChart::class}}" position="c7:d8" />
            </x-dashboard>{%endraw%}
        </div>
    </div>
</div>
@endsection
```

Which didn't work as expected and produced not so good results.

![Preview of paste-in solution](/images/uploads/image-20200703142856250.png)

I then decided to create a new blade template and route for the dashboard, then embed the dashboard as an `iframe` in the homepage, adding custom styles to properly position it.


Here's the new blade template, `resources/views/dashboard.blade.php`:

```html
{%raw%}
<x-dashboard>
    <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExamplePieChart::class}}" position="a1:a2" height="140%"/>
    <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExampleDoughnutChart::class}}" position="b1:b2" height="140%"/>
    <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExamplePieChart::class}}" position="c1:c2" height="140%" />
    <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExampleDoughnutChart::class}}" position="d1:d2" height="140%" />
    <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExampleBarChart::class}}" position="a3:b4" />
    <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExampleLineChart::class}}" position="c3:d4" />
    <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExampleLineChart::class}}" position="a5:b6" />
    <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExampleBarChart::class}}" position="c5:d6" />
    <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExampleBarChart::class}}" position="a7:b8" />
    <livewire:chart-tile chartFactory="{{\Fidum\ChartTile\Examples\ExampleLineChart::class}}" position="c7:d8" />
</x-dashboard>{%endraw%}
```


Here's the new route (that renders the above template):

```php
<?php
    
// ...
    
Route::view('dashboard', 'dashboard')->name('dashboard')->middleware('auth');
```


And here's the updated `home.blade.php` file:

```html
@extends('layouts.app')

@section('content')
    {%raw%}{{-- @see https://stackoverflow.com/a/23027242/6924437 --}}{%endraw%}
    <style>
        html,body {
            height:100%;
            margin-top:0;
            margin-bottom:0;
        }
        .h_iframe iframe {
            width:100%;
            height:100%;
        }
        .h_iframe {
            height: 100vh;
        }
    </style>
<div class="container-fluid">

    <section class="content-header">
        <h1>Dashboard</h1>
    </section>

    <section class="h_iframe">
        <iframe src="{%raw%}{{route('dashboard')}}{%endraw%}" frameborder="0" allowfullscreen></iframe>
    </section>
</div>
@endsection
```

I got the styles from [this](https://stackoverflow.com/a/23027242/6924437) Stackoverflow answer, and had to make some modifications to get it working as expected for me. Hopefully, you don't have to.

Here's what my dashboard now looks like on the home page.

![Preview of final solution with an iframe](/images/uploads/image-20200703144922301.png)

✌️. Please note this is NOT a user-facing app, so am very ok with the iframe solution.
<br>

## Further Reading

* [How to set the iframe height & width to 100% - Stackoverflow](https://stackoverflow.com/questions/23027068/how-to-set-the-iframe-height-width-to-100)
* [A first look at the laravel-dashboard package - FREEK.DEV](https://freek.dev/1643-a-first-look-at-the-laravel-dashboard-package)