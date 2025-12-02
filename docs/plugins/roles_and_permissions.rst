Roles and Permissions
######################

Mautic provides a permission system that defines what each Role can access or modify. Permissions apply across different areas of the application and control which actions are available to Users.

How Permissions Work
********************

Mautic represents permissions using bit values. Each bit doubles in value as it increases, for example:

``1, 2, 4, 8, 16, 32, 64, 128``

Always follow this sequence. Values such as ``3`` or ``5`` cause permission checks to fail as the permission will not correctly calculated.

Example permission set:

+--------------+-----+
| Permission   | Bit |
+--------------+-----+
| view         | 1   |
| edit         | 2   |
| create       | 4   |
| delete       | 8   |
| full         | 16  |
+--------------+-----+

A permission notion follows this format:

``plugin:helloWorld:worlds:view``

This checks the ``view`` permission for the ``worlds`` level of a Plugin.

Bit Storage
===========

Mautic stores permissions by adding the bits of all permissions assigned to a Role. For example:

* ``view`` + ``edit`` → ``1 + 2 = 3``
* ``view`` + ``create`` → ``1 + 4 = 5``

Access checks confirm whether the required bit is present within the stored sum. The ``full`` permission always uses the highest bit and automatically grants all lower permissions.

Using Permissions
-----------------

Use the Security service to check permissions.

Example in Twig:

.. code-block:: twig

   {% if security.isGranted('user:roles:edit') %}
       {# User can edit roles #}
   {% endif %}

Permission notation:

* Core bundles: ``bundle:level:permission``
* Plugins: ``plugin:bundle:level:permission``

Example:

``user:roles:view``

Creating Custom Permissions
---------------------------

Plugins can define custom Permission classes. Each Permission class must:

* Extend ``Mautic\CoreBundle\Security\Permissions\AbstractPermissions``
* Implement ``__construct()``
* Implement ``buildForm()``
* Implement ``getName()``

Constructor
~~~~~~~~~~~

The constructor must call ``parent::__construct($params)`` or assign ``$this->params = $params``.
It must also define ``$this->permissions`` as an array of permission levels and their bit values.

Example level definition:

* Level: ``worlds``
* Permissions: ``use_telescope``, ``send_probe``, ``visit``, ``full``

Access check example:

``plugin:helloWorld:worlds:send_probe``

Helper Methods for Permission Sets
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Mautic provides helper methods for building permission structures:

* ``addStandardPermissions()`` adds view, edit, create, delete, publish, full
* ``addExtendedPermissions()`` adds creator-based permissions
* ``addManagePermission()`` adds a single manage permission

buildForm()
~~~~~~~~~~~

The ``buildForm()`` method adds permission fields to the Role form.

Available helpers include:

* ``addStandardFormFields()``
* ``addExtendedFormFields()``
* ``addManageFormFields()``

getName()
~~~~~~~~~

This method returns the bundle name in camelCase.

Example:

* Bundle: ``HelloWorldBundle``
* ``getName()`` returns ``helloWorld``
* File name: ``HelloWorldPermissions.php``

Permission Aliases
------------------

Use ``getSynonym()`` to map one permission name to another.

Example:

``editown`` maps to ``edit`` if ``editown`` is not defined.

Analyzing Permissions Before Saving
-----------------------------------

Plugins can modify permissions before saving by implementing:

``analyzePermissions()``

If the method returns ``true``, Mautic runs a second pass and provides ``$isSecondRound = true``.

Advanced Permission Checks
--------------------------

To override bit-based checking, extend:

``isGranted($userPermissions, $name, $level)``

Advanced Support Logic
----------------------

You can customize support checks by overriding:

``isSupported()``

Use this for backward compatibility or custom permission rules.