---
layout: post
hidden: false
title: "Laravel: Dynamic Array Input Form Field Example"
author: Leonel Elimpe
tags:
  - Laravel
last_modified_at: 2020-09-23 11:03:48
---
In my app, I have a promotion's table which has a field called `phone_restrictions`, which is an array of phone numbers a promotion is restricted to.

From my backend admin panel, I'd like to populate this array, edit already existing values, or add new ones. Essentially, make `phone_restrictions` an array input field and add Javascript functionality to dynamically add new inputs to the array, then save to the database. Let's dive in.

![Laravel dynamic array iput fields demo](/images/uploads/laravel-dynamic-array-iput-fields-demo.gif)

### The migration and model setup

`phone_restrictions` is set up as a `json` column on the promotions table.

```php
<?php

// ...

class CreatePromotionsTable extends Migration
{

    public function up()
    {
        Schema::create('promotions', function (Blueprint $table) {
            
            // ...

            // Restrict the promotion to an array of phone numbers
            $table->json( 'phone_restrictions' )->nullable();
            
        });
    }

    // ...
}
```

Here's what the `Promotion` model looks like, only the sections important to this blog post.

```php
<?php

namespace App;

use App\Rules\IsValidCameroonianPhoneNumber;
use Eloquent as Model;

/**
 * Class Promotion
 * @package App
 *
 * @property array $phone_restrictions
 * // ...
 */
class Promotion extends Model
{
    public $fillable = [
        'phone_restrictions',
        
        // ...
    ];

    protected $casts = [
        'phone_restrictions' => 'array',
        
        // ...
    ];

    public static function rules() {
        return [
            
            // ...
            
            'phone_restrictions' => [
                'nullable',
                'array',
            ],
            'phone_restrictions.*' => [
                'nullable',
                new IsValidCameroonianPhoneNumber,
            ],
        ];
    }
}
```

`phone_restrictions` is cast to an array - the json value pulled from the database is cast to an array, and there are Request validation rules to make sure if `phone_restrictions` [is not null](https://www.leonelngande.com/laravel-validation-sometimes-vs-nullable/), it should be an array. There's equally validation for each value in the array, if the value is not null, it should be a valid phone number.

### The view setup

P.S., my project uses the [Laravel Collective](https://github.com/laravelcollective/html) package.

I have a `views/promotions/fields.blade.php` partial that contains the core promotion creation and editing form fields as seen below (only the parts related to the `phone_restrictions` input). 

```phtml
<!-- Phone Restrictions [] Field -->
<div class="col-sm-12">
    <br>
    <fieldset>
        <legend>Phone Restrictions</legend>
        <div class="row">
            <div class="input_fields_wrap">
                <!--  @see https://gist.github.com/whoisryosuke/04731177e772fefe08929e809f4fef05#gistcomment-3256172 -->
                @foreach(old('phone_restrictions', isset($promotion) ? $promotion->phone_restrictions : []) as $key => $item)
                    <div class="form-group col-sm-2">
                        <input class="form-control" name="phone_restrictions[]" type="text" value="{{$item}}">
                    </div>
                @endforeach

            </div>

            <div class="col-sm-12">
                <button class="btn btn-link btn-sm add_field_button" type="button">+ Add</button>
            </div>

        </div>
    </fieldset>
    <br>
</div>

@push('scripts')
    <script type="text/javascript">
        
        <!--    @see https://stackoverflow.com/a/36973538/6924437    -->
          
        $(document).ready(function() {
            var wrapper         = $(".input_fields_wrap"); //Fields wrapper
            var add_button      = $(".add_field_button"); //Add button ID

            $(add_button).click(function(e){ //on add input button click
                e.preventDefault();
                //add input box
                var template = `<div class="form-group col-sm-2"> <input class="form-control" name="phone_restrictions[]" type="text"></div>`;
                
                $(wrapper).append(template);
            });
        });
    </script>
@endpush


<!-- Submit Field -->
<div class="form-group col-sm-12">
    {!! Form::submit('Save', ['class' => 'btn btn-primary']) !!}
</div>
```

I believe the line `@foreach(old('phone_restrictions', isset($promotion) ? $promotion->phone_restrictions : []) as $key => $item)` deserves to be explained.

Given some input fields will be added dynamically using Javascript, should a validation error occur we need to be able to recreate all previous input fields and fill them with their previous values, that's what the line of code does. We use the `old()` helper to retrieve the old phone_restrictions value, [passing in a second parameter](https://gist.github.com/whoisryosuke/04731177e772fefe08929e809f4fef05#gistcomment-3256172) to be returned if there's no old value. This second parameter, `$promotion`, is conditional as it's only available when updating a promotion record. It's retrieved at the level of the controller and passed to the view. When it's unavailable, e.g. when creating a promotion, the ternary operator is setup such that an empty array is returned.

The partial `fields.blade.php` is imported into two views:

* `views/promotions/create.blade.php` for creating new promotion records.

  ```phtml
  @extends('layouts.app')

  @section('content')
      {!! Form::open(['route' => 'promotions.store']) !!}

  		@include('promotions.fields')

  	{!! Form::close() !!}
  @endsection
  ```
* `views/promotions/edit.blade.php` for updating promotion records.

  ```phtml
  @extends('layouts.app')

  @section('content')
  	{!! Form::model($promotion, ['route' => ['promotions.update', $promotion->id], 'method' => 'patch']) !!}

  		@include('promotions.fields')

  	{!! Form::close() !!}
  @endsection
  ```

These views push the form data to the `store` and `update` controller methods on submit.

### The controller setup

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests;
use App\Http\Requests\CreatePromotionRequest;
use App\Http\Requests\UpdatePromotionRequest;
use App\Promotion;
use Flash;
use Response;

class PromotionController extends AppBaseController
{
    /**
     * Show the form for creating a new Promotion.
     *
     * @return Response
     */
    public function create()
    {
        return view('promotions.create');
    }

    /**
     * Store a newly created Promotion in storage.
     *
     * @param CreatePromotionRequest $request
     *
     * @return Response
     */
    public function store(CreatePromotionRequest $request)
    {
        $input = $request->all();

        // Remove empty strings in phone_restrictions array, remove repeated numbers
        if (!empty($input['phone_restrictions'])) {
            $input['phone_restrictions'] = array_filter($input['phone_restrictions'], static function ($item) {
                return !empty($item);
            });

            $input['phone_restrictions'] = array_unique($input['phone_restrictions']);
        }

        /** @var Promotion $promotion */
        $promotion = Promotion::create($input);

        Flash::success('Promotion saved successfully.');

        return redirect(route('promotions.index'));
    }

    /**
     * Show the form for editing the specified Promotion.
     *
     * @param  int $id
     *
     * @return Response
     */
    public function edit($id)
    {
        /** @var Promotion $promotion */
        $promotion = Promotion::find($id);

        if ($promotion === null) {
            Flash::error('Promotion not found');

            return redirect(route('promotions.index'));
        }

        return view('promotions.edit', compact(['promotion']));
    }

    /**
     * Update the specified Promotion in storage.
     *
     * @param  int              $id
     * @param UpdatePromotionRequest $request
     *
     * @return Response
     */
    public function update($id, UpdatePromotionRequest $request)
    {
        /** @var Promotion $promotion */
        $promotion = Promotion::find($id);

        if ($promotion === null) {
            Flash::error('Promotion not found');

            return redirect(route('promotions.index'));
        }

        $input = $request->all();

        // Remove empty strings in phone_restrictions array, remove repeated numbers
        if (!empty($input['phone_restrictions'])) {
            $input['phone_restrictions'] = array_filter($input['phone_restrictions'], static function ($item) {
                return !empty($item);
            });

            $input['phone_restrictions'] = array_unique($input['phone_restrictions']);
        }

        $promotion->fill($input);
        $promotion->save();

        Flash::success('Promotion updated successfully.');

        return redirect(route('promotions.index'));
    }

}
```

The `phone_restrictions` array is cleaned up before saving in both the `store` and `update` methods. Given the ability to remove empty fields is missing at the view layer, we need to make all empty fields (containing `null` values) are filtered out. We also make sure no two fields contain the same phone number by passing the array through PHP's `array_unique` function.

Here's what the final result looks like:

![Laravel dynamic array iput fields demo](/images/uploads/laravel-dynamic-array-iput-fields-demo.gif)

That's it for this post, if you've found a typo, issue, or have any questions, please leave a comment below and I'll get back to you.