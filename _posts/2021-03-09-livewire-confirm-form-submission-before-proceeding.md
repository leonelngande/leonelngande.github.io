---
layout: post
hidden: false
title: "Livewire: Confirm form submission before proceeding"
author: Leonel Elimpe
tags:
  - Livewire
last_modified_at: 2021-03-09 06:39:20
---
Spent the last couple of hours trying to get this working. Here's what finally worked:

```phtml
<div>

    <form wire:submit.prevent="submit">

        <div class="form-group">
            
            <input wire:model="username"
                   type="text"
                   name="username"
                   placeholder="Username"
                   value="{% raw %}{{ old('username') }}{% endraw %}"
                   class="form-control" />

            @error('username') <span class="text-danger">{{ $message }}</span> @enderror
        </div>

        <button onclick="confirm('Are you sure?!') || event.preventDefault();"
                type="submit">
            Submit
        </button>

    </form>
    
</div>
```

The most relevant piece being the addition of `onclick="confirm('Are you sure?!') || event.preventDefault();"` to the submit button. Also notice the user of `event.preventDefault()` instead of `event.stopImmediatePropagation()` as you probably might have seen in many blog posts as well as the Livewire documentation. Something like this:

```html
<button onclick="confirm('Are you sure?!') || event.stopImmediatePropagation();"
        type="submit">
    Submit
</button>
```

For the use case of confirming a form submission, `event.stopImmediatePropagation()` doesn't stop the submit action even if the user does not confirm the operation. `event.preventDefault()` does the trick.

`event.stopImmediatePropagation()` however works perfectly when you are only handling a click event on the button, no submit event.