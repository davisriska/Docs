# Form Maker

Our Form Maker package is designed to give developers to freedom to build forms via classes and set the fields in those classes comfortably. You can easily pass a form to a view and it will render the whole form, reducing your need to type out divs, labels, inputs and more.

[Source Code](https://github.com/grafiteinc/formMaker)

!!! warning "Form Maker by default is tightly set to Bootstrap's classes."

The form maker

## Artisan Commands

Generate a form for a specific model using this make command. It will add the ModelForm to a Forms directory in the `app/Html/Forms` namespace.
```
make:model-form {model}
```

Generate a generic form with a specific name using this command. It will add the BaseForm to a Forms directory in the `app/Html/Forms` namespace.
```
make:base-form {name}
```

The following command will generate a feature test which hits the endpoints of the form.

```
make:form-test {form}
```

If you made a ModelForm you likely want a factory for your model, since we have the fields within the ModelForm class we can generate basic factories.

```
make:form-factory {form}
```

## Set Alternate Connections

```
app(UserForm::class)->setConnection('alternate');
```

Or in the `UserForm` itself:

```
$connection = 'alternate';
```

## Helpers

```
form() // access the `Form` class
```

## Fields

Fields are PHP objects which define types, options, attributes etc for the Form Object. This lets you design forms and update them easily via a single object rather than having to update HTML files for multiple roles in your application.

```
FIELD_OPTIONS
type: A type string such as text or file
options: Options for <select> type
legend: A label for the legend of horizontal checkboxes
null_value: False by default, but allows empty values to be placed at the front of selects
null_label: Text which is placed as the first option in a select with a blank value
label: A label
model: Model class for the HasOne and HasMany Fields
model_options: Model options
    label: The label attribute on the model
    value: The value attribute on the model
    method: A custom method to run on the model
    params: Parameters to send to the custom method
before: Text or HTML you wish to put before an input
after: Text or HTML you wish to put after the input
view: A view path to a custom template
attributes: HTML attributes for your input field
format: Format for DateTime objects
```

There are a large collection of Fields available out of the box and a `make:field {name}` command in case you need custom ones. Fields generate a config array which is then processed by the `FieldMaker` and `FieldBuilder` to create forms for easy use.

#### Available Fields

```
Checkbox
CheckboxInline
Color
CustomFile
Date
DatetimeLocal
Decimal
Email
File
HasMany
HasOne
Hidden
Image
Month
Number
Password
Radio
RadioInline
Range
Telephone
Text
TextArea
Time
Url
Week
```

## Sections

The various Form Objects allow you to set Sections. For example, you may have a `BlogForm` and you may want one row to have three columns while the next row has two, this can be achieved with the `setSections` method.

```
$columns = 1;

public function fields()
{
    return [
        Text::make('title', [
            'required' => true,
        ]),
        Text::make('url', [
            'required' => true
        ]),
        TextArea::make('entry', []),
        Date::make('published_at', []),
    ];
}
```

By default this will build a form with single column content. If you wish to set these fields to specific layouts you need to set the `columns` to `sections`

```
$columns = 'sections';

public function fields()
{
    return [
        Text::make('title', [
            'required' => true,
        ]),
        Text::make('url', [
            'required' => true
        ]),
        TextArea::make('entry', []),
        Date::make('published_at', []),
    ];
}

public function setSections()
{
    return [
        [
            'title',
            'url',
        ],
        [
            'entry'
        ],
        [
            'published_at'
        ]
    ];
}
```

The above will produce a form that is two columns, one, and one. You can add a `key` to the sections to add a horizontal line and a header.

```
public function setSections()
{
    return [
        [
            'title',
            'url',
        ],
        'Content' => [
            'entry'
        ],
        [
            'published_at'
        ]
    ];
}
```

## Form

The Form class lets us generate simple forms with minimal code.

```
form()->action('method', 'route', 'button text', $html_attributes);
```

Generates a form using the method and route with a button, for easier addition of delete buttons and more,

```
form()->confirm('Are you sure?')->action(...);
```
Adds a confirmation popup to the button.

!!! warning "confirm is only supported in the delete method of ModelForms."

If you wish to handle the confirm using a modal or other JS integration you can pass a `method` name into the confirm method which will trigger that JS method:

```
form()->confirm('Are you sure you want to delete this?', 'confirmation')->action(...);
```

This will add the following to the submit button in the form:

```
onclick="confirmation(event, 'Are you sure you want to delete this?')"
```

You will require a `confirmation` method to be defined somewhere such as in `app.js`. Your confirmation method could look something like this, which uses Bootstrap's modal:

```
window.confirmation = (_event, _message) => {
    _event.preventDefault();

    $('#appModalMessage').html(_message);

    $('#appModal').modal({
        show: true
    });

    $('#appModalConfirmBtn').click(() => {
        $(_event.target.parentNode)[0].submit();
    });

    return false;
}
```

There is also support for an `submitMethod` attribute on all forms. This custom submit allows you to override the standard submit and perform actions against your form. This can be useful if you want to do an ajax submission of your form.

```
<?php

namespace App\Http\Forms;

use Grafite\FormMaker\Fields\Password;
use Grafite\FormMaker\Forms\BaseForm;

class UserSecurityForm extends BaseForm
{
    public $route = 'user.security';

    public $method = 'put';

    public $orientation = 'horizontal';

    public $submitMethod = 'ajax';

    public $buttons = [
        'submit' => 'Save',
    ];

    public $columns = 1;

    public function fields()
    {
        return [
            Password::make('old_password', [
                'required' => true,
                'label' => 'Old Password'
            ]),
            Password::make('new_password', [
                'required' => true,
                'label' => 'New Password'
            ]),
            Password::make('new_password_confirmation', [
                'required' => true,
                'label' => 'Confirm New Password'
            ]),
        ];
    }
}
```

```
window.ajax = (_event) => {
    _event.preventDefault();

    let _form = _event.target.parentNode.parentNode.parentNode;
    let _method = _form.method.toLowerCase();
    let _payloadArray = $(_form).serializeArray();
    let _payload = {};

    $.map(_payloadArray, function(n, i){
        _payload[n['name']] = n['value'];
    });

    window.axios[_method](_form.action, _payload)
        .then((response) => {
            window.Snotify.success(response.data.message);
        })
        .catch((response) => {
            window.Snotify.warning(response.data.message);
        });
}
```

```
form()->open($options);
```

Opens a form allowing you to specify options: action, method, attributes etc.

```
form()->model($model, $options);
```

Open a form based on a model

---

## ModelForm

Using the `make:model-form {model}` command you can quickly generate forms for Models. This will let you generate forms based on the model.

```
app(UserForm::class)->create();
app(UserForm::class)->edit($user);
app(UserForm::class)->delete($user);
```

### Example

```
<?php

namespace App\Http\Forms;

use App\Models\Role;
use App\Models\User;
use Grafite\FormMaker\Fields\File;
use Grafite\FormMaker\Fields\Text;
use Grafite\FormMaker\Fields\Email;
use Grafite\FormMaker\Fields\Checkbox;
use Grafite\FormMaker\Forms\ModelForm;

class UserForm extends ModelForm
{
    public $model = User::class;

    public $routePrefix = 'user';

    public $buttons = [
        'submit' => 'Save',
    ];

    public $columns = 1;

    public $orientation = 'horizontal';

    public $hasFiles = true;

    public function fields()
    {
        return [
            Text::make('name', [
                'required' => true,
            ]),
            Email::make('email', [
                'required' => true
            ]),
            Checkbox::make('dark_mode', [
                'legend' => 'Dark Mode'
            ]),
            File::make('avatar', []),
        ];
    }
}
```

Within this `UserForm` class you can set the fields in in the `fields` method:

```
public function fields()
{
    return [
        Text::make('name', [
            'required' => true,
        ]),
        Email::make('email', [
            'required' => true
        ]),
    ];
}
```

This will generate a form with these fields only. You can also set the `orientation` if you wish to use labels on the left side instead of above and `columns` if you wish to generate a form in which the fields are split into more columns.

`$routePrefix` is required as it defines the routes using route names which match the standards route naming of Laravel.

`$hasFiles` enables the file submission of the form.

`$buttons` enables you to customize the button values for submit and cancel

With the `delete()` form you can also add the confirmation method like so:

```
{!! form()
    ->confirm('Are you sure you want to delete your avatar?', 'confirmation')
    ->action('delete', 'user.destroy.avatar', 'delete', ['class' => 'btn btn-sm btn-outline-secondary'])
!!}
```

### Custom View for Fields

You can create a custom view that the FormMaker will use for your fields. Just have you view file follow this pattern:

```
<div class="row">
    <div class="form-group">
        <label class="control-label" for="{!! $labelFor !!}">{!! $label !!}</label>
        <div class="row">
            {!! $field !!}
        </div>
    </div>
    {!! $errors !!}
</div>
```

### Relationships

FormMaker also has Fields called `HasOne` and `HasMany`. These enable you to select one or multiple from a relationship binding in your model.

For example you may have a `User` who has a `role` which is `belongsToMany`.

You can set the Field for something like 'Roles' as below.

```
HasOne::make('role', [
    'model' => Role::class,
    'model_options' => [
        'label' => 'name',
        'value' => 'id',
        'method' => 'all',
        'params' => null,
    ]
]);
```

You can also customize how you access the method and with what parameters. By default it will try to collect the options by getting `Role::all()` however you can limit that with something like `Role::custom(params)`.
