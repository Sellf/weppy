Forms
=====

weppy provides the `Form` class to let you easily create forms for your application.

Let's see how to use it with an example:

```python
from weppy import Field, Form

# create a form
@app.route('/form')
def a():
    simple_form = Form({
        'name': Field(),
        'number': Field('int'),
        'type': Field(
            validation={'in': ['type1', 'type2']}
        )
    })
    if simple_form.accepted:
        inserted_number = form.params.number
        #do something
    return dict(form=simple_form)
```

As you can see, the `Form` class accepts a `dict` of `Field` objects as input,
and those are described in the [DAL chapter](./dal#fields) of the documentation.
Forms validate the input of the clients using their fields' validation: when the
input passes the validation, the `accepted` attribute is set to `True`.
The example above shows you that you can use this attribute to do things when
clients submit the form, and the submitted values are stored in `form.params`.

Forms with DAL entities
-----------------------
Forms are quite handy for inserting or editing data in your database, for this purpose
weppy provides another class: `DALForm`. The usage is the same of the form,
except that you call it directly from your model:

```python
# create a form for Post model
@app.route('/dalform')
def b():
    form = Post.form()
    if form.accepted:
        #do something
    return dict(form=form)
```

where, obviously, the `form()` method of the models is a shortcut for the `DALForm` class.

> – Wait, what if I need to edit a record?

You can pass the record as an argument in `Model.form()`:

```python
record = db.Post(id=1)
form = Post.form(record)
```

If you prefer, you can also use a record id:

```python
form = Post.form(record_id=1)
```

Here is the complete list of parameters accepted by `Form` class:

| parameter | default | description |
| --- | --- | --- |
| _action | `None` | allows you to set the HTML `action` tag of the form |
| _method | `'POST'` | set the form submit method (GET or POST) |
| _enctype | `'multipart/form-data'` | allows you to change the encoding type for the submitted data |
| submit | `'Submit'` | the text to show in the submit button |
| formstyle | `FormStyle` | the class used to style the form |
| csrf | `'auto'` | Cross-Site Request Forgery protection |
| keepvalues | `False` | set if the form should keep the values in the input fields after submit |
| id_prefix | `None` | allows you to set a prefix for the id of the form fields |
| onvalidation | `None` | set an additional validation for the form |
| upload | `None` | define a URL for download uploaded fields |

`DALForm` class add some parameters to the `Form` ones:

| parameter | description |
| --- | --- |
| record | as we seen above, set a record to edit |
| record_id | alternative to `record` using id |
| fields | list of fields (names) to show in the record |
| exclude_fields | list of fields (names) not to be included in the form |

> **Note:** the `fields` and `exclude_fields` parameters should not be used together. If you need to hide just a few fields, you'd better using the `exclude_fields`, and you should use `fields` if you have to show only few table fields. The advantages of these parameters are lost if you use both.

###Uploads with forms
As we saw above, the `upload` parameter of forms needs an URL for download. 
Let's focus a bit on uploads and see an example to completely understand this requirement.

Let's say you want to handle the upload of avatar images from your user. So,
in your model/table, you would have an upload field:

```python
avatar = Field('upload')
```

and the forms produced by weppy will handle uploads for you. How would you
display this image in your template? You need a streaming function like this:

```
from weppy import stream_file 

@app.route("/download/<str:filename>")
def download(filename):
    stream_file(db, filename)
```

and then, in your template, you can create an `img` tag pointing to the
`download` function you've just exposed:

```html
<img src="{{=url('download', record.avatar')}}" />
```

The `upload` parameter of `Form` class has the same purpose: when you edit an
existent record the form will display the image or file link for the existing
one uploaded. In this example you would do:

```python
record = db.Post(id=someid)
form = Post.form(record, upload=url('download'))
```

Custom validation
-----------------
The `onvalidation` parameter of forms allows you to add custom validation logic
to your form. You can pass a callable function, and it will be invoked after the
form has processed the fields validators. This means that your function will be
invoked only if there weren't errors with the fields validators.

Let's see what we're talking about with an example:

```python
@app.route("/myform")
def myform():
    def process_form(form):
        if form.params.double != form.params.number*2:
            form.errors.double = "Double is incorrect!"
    
    form = Form(
        number=Field('int'), 
        double=Field('int'),
        onvalidation=process_form
    )
    return dict(form=form)
```

where the form checks if the second number is double the first and returns
an error if the input is wrong.

You've just learned how to use the `onvalidation` parameter and that you can
store errors in `form.errors`, which is a `sdict` object like `form.params`.

Also, you understood that `Form` also accepts `Field` objects as arguments.

Customizing forms
-----------------
Good applications also need good styles. This is why weppy forms allows you to
set a specific style with the `formstyle` attribute. But how should you edit the
style of your form?

Well, in weppy, the style of a form is decided by the `FormStyle` class.

###Creating your style
*sub-section under development*

###Custom widgets
*sub-section under development*
