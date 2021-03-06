Modelless Forms
###############

.. php:namespace:: Cake\Form

.. php:class:: Form

Most of the time you will have forms backed by :doc:`ORM entities </orm/entities>`
and :doc:`ORM tables </orm/table-objects>` or other peristent stores,
but there are times when you'll need to validate user input and then perform an
action if the data is valid. The most common example of this is a contact form.

Creating a Form
===============

Generally when using the Form class you'll want to use a subclass to define your
form. This makes testing easier, and lets you re-use your form. Forms are put
into ``src/Form`` and usually have ``Form`` as a class suffix. For example,
a simple contact form would look like::

    // in src/Form/ContactForm.php
    namespace App\Form;

    use Cake\Form\Form;
    use Cake\Form\Schema;
    use Cake\Validation\Validator;

    class ContactForm extends Form
    {

        protected function _buildSchema(Schema $schema)
        {
            return $schema->addField('name', 'string')
                ->addField('email', ['type' => 'string'])
                ->addField('body', ['type' => 'text']);
        }

        protected function _buildValidator(Validator $validator)
        {
            return $validator->add('name', 'length', [
                    'rule' => ['minLength', 10],
                    'message' => 'A name is required'
                ])->add('email', 'format', [
                    'rule' => 'email',
                    'message' => 'A valid email address is required',
                ]);
        }

        protected function _execute(array $data)
        {
            // Send an email.
            return true;
        }
    }

In the above example we see the 3 hook methods that forms provide:

* ``_buildSchema`` is used to define the schema data that is used by FormHelper
  to create an HTML form. You can define field type, length, and precision.
* ``_buildValidator`` Gets a :php:class:`Cake\Validation\Validator` instance
  that you can attach validators to.
* ``_execute`` lets you define the behavior you want to happen when
  ``execute()`` is called and the data is valid.

You can always define additional public methods as you need as well.

Processing Request Data
=======================

Once you've defined your form, you can use it in your controller to process
and validate request data::

    // In a controller
    namespace App\Controller;

    use App\Controller\AppController;
    use App\Form\ContactForm;

    class ContactController extends AppController
    {
        public function index()
        {
            $contact = new ContactForm();
            if ($this->request->is('post')) {
                if ($contact->execute($this->request->data)) {
                    $this->Flash->success('We will get back to you soon.');
                } else {
                    $this->Flash->error('There was a problem submitting your form.');
                }
            }
            $this->set('contact', $contact);
        }
    }

In the above example, we use the ``execute()`` method to run our form's
``_execute()`` method only when the data is valid, and set flash messages
accordingly. We could have also used the ``validate()`` method to only validate
the request data::

    $isValid = $form->validate($this->request->data);

Getting Form Errors
===================

Once a form has been validated you can retreive the errors from it::

    $errors = $form->errors();
    /* $errors contains
    [
        'email' => ['A valid email address is required']
    ]
    */

Creating HTML with FormHelper
=============================

Once you've created a Form class, you'll likely want to create an HTML form for
it. FormHelper understands Form objects just like ORM entities::

    echo $this->Form->create($contact);
    echo $this->Form->input('name');
    echo $this->Form->input('email');
    echo $this->Form->input('body');
    echo $this->Form->button('Submit');
    echo $this->Form->end();

The above would create an HTML form for the ``ContactForm`` we defined earlier.
HTML forms created with FormHelper will use the defined schema and validator to
determine field types, maxlengths, and validation errors.
