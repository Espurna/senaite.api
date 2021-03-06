SENAITE API DOCTEST
===================

The SENAITE LIMS API provides single functions for single purposes.
This Test builds completely on the API without any further imports needed.

Running this test from the buildout directory::

    bin/test test_doctests -t API

Introduction
------------

The purpose of this API is to help coders to follow the DRY principle (Don't
Repeat Yourself). It also ensures that the most effective and efficient method is
used to achieve a task.

Import it first::

    >>> from senaite import api


Getting the Portal
------------------

The Portal is the SENAITE LIMS root object::

    >>> portal = api.get_portal()
    >>> portal
    <PloneSite at /plone>


Getting the Bika Setup object
-----------------------------

The Bika Setup object gives access to all of the Bika configuration settings::

    >>> bika_setup = api.get_bika_setup()
    >>> bika_setup
    <BikaSetup at /plone/bika_setup>


Creating new Content
--------------------

Creating new contents in Bika LIMS requires some special knowledge.
This function helps to do it right and creates a content for you.

Here we create a new `Client` in the `plone/clients` folder::

    >>> client = api.create(portal.clients, "Client", title="Test Client")
    >>> client
    <Client at /plone/clients/client-1>

     >>> client.Title()
     'Test Client'


Getting a Tool
--------------

There are many ways to get a tool in Bika LIMS / Plone. This function
centralizes this functionality and makes it painless::

    >>> api.get_tool("bika_setup_catalog")
    <BikaSetupCatalog at /plone/bika_setup_catalog>

Trying to fetch an non-existing tool raises a custom `SenaiteAPIError`.

    >>> api.get_tool("NotExistingTool")
    Traceback (most recent call last):
    [...]
    SenaiteAPIError: No tool named 'NotExistingTool' found.

This error can also be used for custom methods with the `fail` function::

    >>> api.fail("This failed badly")
    Traceback (most recent call last):
    [...]
    SenaiteAPIError: This failed badly


Getting an Object
-----------------

Getting a tool from a catalog brain is a common task in Bika LIMS. This function
provides an unified interface to portal objects **and** brains.
Furthermore it is idempotent, so it can be called multiple times in a row.

We will demonstrate the usage on the client object we created above::

    >>> api.get_object(client)
    <Client at /plone/clients/client-1>

    >>> api.get_object(api.get_object(client))
    <Client at /plone/clients/client-1>

Now we show it with catalog results::

    >>> portal_catalog = api.get_tool("portal_catalog")
    >>> brains = portal_catalog(portal_type="Client")
    >>> brains
    [<Products.ZCatalog.Catalog.mybrains object at 0x...>]

    >>> brain = brains[0]

    >>> api.get_object(brain)
    <Client at /plone/clients/client-1>

    >>> api.get_object(api.get_object(brain))
    <Client at /plone/clients/client-1>

No supported objects raise an error::

    >>> api.get_object(object())
    Traceback (most recent call last):
    [...]
    SenaiteAPIError: <object object at 0x...> is not supported.

To check if an object is supported, e.g. is an ATCT, Dexterity, ZCatalog or
Portal object, we can use the `is_object` function::

    >>> api.is_object(client)
    True

    >>> api.is_object(brain)
    True

    >>> api.is_object(api.get_portal())
    True

    >>> api.is_object(None)
    False

  >>> api.is_object(object())
    False


Checking if an Object is the Portal
-----------------------------------

Sometimes it can be handy to check if the current object is the portal::

    >>> api.is_portal(portal)
    True

    >>> api.is_portal(client)
    False

    >>> api.is_portal(object())
    False


Checking if an Object is a Catalog Brain
----------------------------------------

Knowing if we have an object or a brain can be handy. This function checks this for you::

    >>> api.is_brain(brain)
    True

    >>> api.is_brain(api.get_object(brain))
    False

    >>> api.is_brain(object())
    False


Checking if an Object is a Dexterity Content
--------------------------------------------

This function checks if an object is a `Dexterity` content type::

    >>> api.is_dexterity_content(client)
    False

    >>> api.is_dexterity_content(portal)
    False

We currently have no `Dexterity` contents, so testing this comes later...


Checking if an Object is an AT Content
--------------------------------------

This function checks if an object is an `Archetypes` content type::

    >>> api.is_at_content(client)
    True

    >>> api.is_at_content(portal)
    False

    >>> api.is_at_content(object())
    False


Getting the Schema of a Content
-------------------------------

The schema contains the fields of a content object. Getting the schema is a
common task, but differs between `ATContentType` based objects and `Dexterity`
based objects. This function brings it under one umbrella::

    >>> schema = api.get_schema(client)
    >>> schema
    <Products.Archetypes.Schema.Schema object at 0x...>

Catalog brains are also supported::

    >>> api.get_schema(brain)
    <Products.Archetypes.Schema.Schema object at 0x...>


Getting the Fields of a Content
-------------------------------

The fields contain all the values that an object holds and are therefore
responsible for getting and setting the information.

This function returns the fields as a dictionary mapping of `{"key": value}`::

    >>> fields = api.get_fields(client)
    >>> fields.get("ClientID")
    <Field ClientID(string:rw)>

Catalog brains are also supported::

    >>> api.get_fields(brain).get("ClientID")
    <Field ClientID(string:rw)>


Getting the ID of a Content
---------------------------

Getting the ID is a common task in Bika LIMS.
This function takes care that catalog brains are not waked up for this task::

    >>> api.get_id(portal)
    'plone'

    >>> api.get_id(client)
    'client-1'

    >>> api.get_id(brain)
    'client-1'


Getting the Title of a Content
------------------------------

Getting the Title is a common task in Bika LIMS.
This function takes care that catalog brains are not waked up for this task::

    >>> api.get_title(portal)
    u'Plone site'

    >>> api.get_title(client)
    'Test Client'

    >>> api.get_title(brain)
    'Test Client'


Getting the Description of a Content
------------------------------------

Getting the Description is a common task in Bika LIMS.
This function takes care that catalog brains are not waked up for this task::

    >>> api.get_description(portal)
    ''

    >>> api.get_description(client)
    ''

    >>> api.get_description(brain)
    ''


Getting the UID of a Content
----------------------------

Getting the UID is a common task in Bika LIMS.
This function takes care that catalog brains are not waked up for this task.

The portal object actually has no UID. This funciton defines it therfore to be `0`::

    >>> api.get_uid(portal)
    '0'

    >>> uid_client = api.get_uid(client)
    >>> uid_client_brain = api.get_uid(brain)
    >>> uid_client is uid_client_brain
    True


Getting the URL of a Content
----------------------------

Getting the URL is a common task in Bika LIMS.
This function takes care that catalog brains are not waked up for this task::

    >>> api.get_url(portal)
    'http://nohost/plone'

    >>> api.get_url(client)
    'http://nohost/plone/clients/client-1'

    >>> api.get_url(brain)
    'http://nohost/plone/clients/client-1'


Getting the Icon of a Content
-----------------------------

    >>> api.get_icon(client)
    '<img width="16" height="16" src="http://nohost/plone/++resource++bika.lims.images/client.png" title="Test Client" />'

    >>> api.get_icon(brain)
    '<img width="16" height="16" src="http://nohost/plone/++resource++bika.lims.images/client.png" title="Test Client" />'

    >>> api.get_icon(client, html_tag=False)
    'http://nohost/plone/++resource++bika.lims.images/client.png'

    >>> api.get_icon(client, html_tag=False)
    'http://nohost/plone/++resource++bika.lims.images/client.png'


Getting an object by UID
------------------------

This function finds an object by its uinique ID (UID).
The portal object with the defined UId of '0' is also supported::

    >>> api.get_object_by_uid('0')
    <PloneSite at /plone>

    >>> api.get_object_by_uid(uid_client)
    <Client at /plone/clients/client-1>

    >>> api.get_object_by_uid(uid_client_brain)
    <Client at /plone/clients/client-1>

If a default value is provided, the function will never fail.  Any exception
or error will result in the default value being returned::

    >>> api.get_object_by_uid('invalid uid', 'default')
    'default'

    >>> api.get_object_by_uid(None, 'default')
    'default'


Getting an object by Path
-------------------------

This function finds an object by its physical path::

    >>> api.get_object_by_path('/plone')
    <PloneSite at /plone>

    >>> api.get_object_by_path('/plone/clients/client-1')
    <Client at /plone/clients/client-1>

Paths outside the portal raise an error::

    >>> api.get_object_by_path('/root')
    Traceback (most recent call last):
    [...]
    SenaiteAPIError: Not a physical path inside the portal.

Any exception returns default value::

    >>> api.get_object_by_path('/invaid/path', 'default')
    'default'

    >>> api.get_object_by_path(None, 'default')
    'default'


Getting the Physical Path of an Object
--------------------------------------

The physical path describes exactly where an object is located inside the portal.
This function unifies the different approaches to get the physical path and does
so in the most efficient way::

    >>> api.get_path(portal)
    '/plone'

    >>> api.get_path(client)
    '/plone/clients/client-1'

    >>> api.get_path(brain)
    '/plone/clients/client-1'

    >>> api.get_path(object())
    Traceback (most recent call last):
    [...]
    SenaiteAPIError: <object object at 0x...> is not supported.


Getting the Physical Parent Path of an Object
---------------------------------------------

This function returns the physical path of the parent object::

    >>> api.get_parent_path(client)
    '/plone/clients'

    >>> api.get_parent_path(brain)
    '/plone/clients'

However, this function goes only up to the portal object::

    >>> api.get_parent_path(portal)
    '/plone'

Like with the other functions, only portal objects are supported::

    >>> api.get_parent_path(object())
    Traceback (most recent call last):
    [...]
    SenaiteAPIError: <object object at 0x...> is not supported.


Getting the Parent Object
-------------------------

This function returns the parent object::

    >>> api.get_parent(client)
    <ClientFolder at /plone/clients>

Brains are also supported::

    >>> api.get_parent(brain)
    <ClientFolder at /plone/clients>

The function can also use a catalog query on the `portal_catalog` and return a
brain, if the passed parameter `catalog_search` was set to true. ::

    >>> api.get_parent(client, catalog_search=True)
    <Products.ZCatalog.Catalog.mybrains object at 0x...>

    >>> api.get_parent(brain, catalog_search=True)
    <Products.ZCatalog.Catalog.mybrains object at 0x...>

However, this function goes only up to the portal object::

    >>> api.get_parent(portal)
    <PloneSite at /plone>

Like with the other functions, only portal objects are supported::

    >>> api.get_parent(object())
    Traceback (most recent call last):
    [...]
    SenaiteAPIError: <object object at 0x...> is not supported.


Searching Objects
-----------------

Searching in Bika LIMS requires knowledge in which catalog the object is indexed.
This function unifies all Bika LIMS catalog to a single search interface::

    >>> results = api.search({'portal_type': 'Client'})
    >>> results
    [<Products.ZCatalog.Catalog.mybrains object at 0x...>]

Multiple content types are also supported::

    >>> results = api.search({'portal_type': ['Client', 'ClientFolder'], 'sort_on': 'getId'})
    >>> map(api.get_id, results)
    ['client-1', 'clients']

Now we create some objects which are located in the `bika_setup_catalog`::

    >>> instruments = bika_setup.bika_instruments
    >>> instrument1 = api.create(instruments, "Instrument", title="Instrument-1")
    >>> instrument2 = api.create(instruments, "Instrument", title="Instrument-2")
    >>> instrument3 = api.create(instruments, "Instrument", title="Instrument-3")

    >>> results = api.search({'portal_type': 'Instrument', 'sort_on': 'getId'})
    >>> len(results)
    3

    >>> map(api.get_id, results)
    ['instrument-1', 'instrument-2', 'instrument-3']

Queries which result in multiple catalogs will be refused, as it would require
manual merging and sorting of the results afterwards. Thus, we fail here:

    >>> results = api.search({'portal_type': ['Client', 'ClientFolder', 'Instrument'], 'sort_on': 'getId'})
    Traceback (most recent call last):
    [...]
    SenaiteAPIError: Multi Catalog Queries are not supported, please specify a catalog.

Catalog queries w/o any `portal_type`, default to the `portal_catalog`, which
will not find the following items::

    >>> analysiscategories = bika_setup.bika_analysiscategories
    >>> analysiscategory1 = api.create(analysiscategories, "AnalysisCategory", title="AC-1")
    >>> analysiscategory2 = api.create(analysiscategories, "AnalysisCategory", title="AC-2")
    >>> analysiscategory3 = api.create(analysiscategories, "AnalysisCategory", title="AC-3")

    >>> results = api.search({"id": "analysiscategory-1"})
    >>> len(results)
    0

Would we add the `portal_type`, the search function would ask the
`archetype_tool` for the right catalog, and it would return a result::

    >>> results = api.search({"portal_type": "AnalysisCategory", "id": "analysiscategory-1"})
    >>> len(results)
    1

We could also explicitly define a catalog to achieve the same::

    >>> results = api.search({"id": "analysiscategory-1"}, catalog="bika_setup_catalog")
    >>> len(results)
    1

To see inactive or dormant items, we must explicitly query them of filter them
afterwars manually::

    >>> results = api.search({"portal_type": "AnalysisCategory", "id": "analysiscategory-1"})
    >>> len(results)
    1

Now we deactivate the item::

    >>> analysiscategory1 = api.do_transition_for(analysiscategory1, 'deactivate')
    >>> api.is_active(analysiscategory1)
    False

The search will still find the item::

    >>> results = api.search({"portal_type": "AnalysisCategory", "id": "analysiscategory-1"})
    >>> len(results)
    1

Unless we filter it out manually::

    >>> len(filter(api.is_active, results))
    0

Or provide a correct query::

    >>> results = api.search({"portal_type": "AnalysisCategory", "id": "analysiscategory-1", "inactive_status": "active"})
    >>> len(results)
    1


Getting the registered Catalogs
-------------------------------

Bika LIMS uses multiple catalogs registered via the Archetype Tool. This
function returns a list of registered catalogs for a brain or object::

    >>> api.get_catalogs_for(client)
    [<CatalogTool at /plone/portal_catalog>]

    >>> api.get_catalogs_for(instrument1)
    [<BikaSetupCatalog at /plone/bika_setup_catalog>, <CatalogTool at /plone/portal_catalog>]

    >>> api.get_catalogs_for(analysiscategory1)
    [<BikaSetupCatalog at /plone/bika_setup_catalog>]


Getting an Attribute of an Object
---------------------------------

This function handles attributes and methods the same and returns their value.
It also handles security and is able to return a default value instead of
raising an `Unauthorized` error::

    >>> uid_brain = api.safe_getattr(brain, "UID")
    >>> uid_obj = api.safe_getattr(client, "UID")

    >>> uid_brain == uid_obj
    True

    >>> api.safe_getattr(brain, "review_state")
    'active'

    >>> api.safe_getattr(brain, "NONEXISTING")
    Traceback (most recent call last):
    [...]
    SenaiteAPIError: Attribute 'NONEXISTING' not found.

    >>> api.safe_getattr(brain, "NONEXISTING", "")
    ''

Getting the Portal Catalog
--------------------------

This tool is needed so often, that this function just returns it::

    >>> api.get_portal_catalog()
    <CatalogTool at /plone/portal_catalog>


Getting the Review History of an Object
---------------------------------------

The review history gives information about the objects' workflow changes::

    >>> review_history = api.get_review_history(client)
    >>> sorted(review_history[0].items())
    [('action', None), ('actor', 'test_user_1_'), ('comments', ''), ('review_state', 'active'), ('time', DateTime('...'))]


Getting the Revision History of an Object
-----------------------------------------

The review history gives information about the objects' workflow changes::

    >>> revision_history = api.get_revision_history(client)
    >>> sorted(revision_history[0])
    ['action', 'actor', 'actor_home', 'actorid', 'comments', 'review_state', 'state_title', 'time', 'transition_title', 'type']
    >>> revision_history[0]["transition_title"]
    u'Create'


Getting the assigned Workflows of an Object
-------------------------------------------

This function returns all assigned workflows for a given object::

    >>> api.get_workflows_for(bika_setup)
    ('bika_one_state_workflow',)

    >>> api.get_workflows_for(client)
    ('bika_client_workflow', 'bika_inactive_workflow')

This function also supports the portal_type as parameter::

    >>> api.get_workflows_for(api.get_portal_type(client))
    ('bika_client_workflow', 'bika_inactive_workflow')


Getting the Workflow Status of an Object
----------------------------------------

This function returns the state of a given object::

    >>> api.get_workflow_status_of(client)
    'active'

It is also capable to get the state of another state variable::

    >>> api.get_workflow_status_of(client, "inactive_state")
    'active'

Deactivate the client::

    >>> api.do_transition_for(client, "deactivate")
    <Client at /plone/clients/client-1>

    >>> api.get_workflow_status_of(client, "inactive_state")
    'inactive'

    >>> api.get_workflow_status_of(client)
    'active'

Reactivate the client::

    >>> api.do_transition_for(client, "activate")
    <Client at /plone/clients/client-1>

    >>> api.get_workflow_status_of(client, "inactive_state")
    'active'


Getting the registered Catalogs of an Object
--------------------------------------------

This function returns a list of all registered catalogs within the
`archetype_tool` for a given portal_type or object::

    >>> api.get_catalogs_for(client)
    [<CatalogTool at /plone/portal_catalog>]

It also supports the `portal_type` as a parameter::

    >>> api.get_catalogs_for("Analysis")
    [<BikaAnalysisCatalog at /plone/bika_analysis_catalog>]


Transitioning an Object
-----------------------

This function performs a workflow transition and returns the object::

    >>> client = api.do_transition_for(client, "deactivate")
    >>> api.is_active(client)
    False

    >>> client = api.do_transition_for(client, "activate")
    >>> api.is_active(client)
    True


Getting inactive/cancellation state of different workflows
----------------------------------------------------------

There are two workflows allowing an object to be set inactive.  We provide
the is_active function to return False if an item is set inactive with either
of these workflows.

In the search() test above, the is_active function's handling of brain states
is tested.  Here, I just want to test if object states are handled correctly.

For setup types, we use bika_inctive_workflow::

    >>> method1 = api.create(portal.methods, "Method", title="Test Method")
    >>> api.is_active(method1)
    True
    >>> method1 = api.do_transition_for(method1, 'deactivate')
    >>> api.is_active(method1)
    False

For transactional types, bika_cancellation_workflow is used::

    >>> batch1 = api.create(portal.batches, "Batch", title="Test Batch")
    >>> api.is_active(batch1)
    True
    >>> batch1 = api.do_transition_for(batch1, 'cancel')
    >>> api.is_active(batch1)
    False


Getting the granted Roles for a certain Permission on an Object
---------------------------------------------------------------

This function returns a list of Roles, which are granted the given Permission
for the passed in object::

    >>> api.get_roles_for_permission("Modify portal content", bika_setup)
    ['LabManager', 'Manager']



Checking if an Object is Versionable
------------------------------------

Some contents in Bika LIMS support versioning. This function checks this for you.

Instruments are not versionable::

    >>> api.is_versionable(instrument1)
    False

Analysisservices are versionable::

    >>> analysisservices = bika_setup.bika_analysisservices
    >>> analysisservice1 = api.create(analysisservices, "AnalysisService", title="AnalysisService-1")
    >>> analysisservice2 = api.create(analysisservices, "AnalysisService", title="AnalysisService-2")
    >>> analysisservice3 = api.create(analysisservices, "AnalysisService", title="AnalysisService-3")

    >>> api.is_versionable(analysisservice1)
    True


Getting the Version of an Object
--------------------------------

This function returns the version as an integer::

    >>> api.get_version(analysisservice1)
    0

Calling `processForm` bumps the version::

    >>> analysisservice1.processForm()
    >>> api.get_version(analysisservice1)
    1


Getting a Browser View
----------------------

Getting a browser view is a common task in Bika LIMS::

    >>> api.get_view("plone")
    <Products.Five.metaclass.Plone object at 0x...>

    >>> api.get_view("workflow_action")
    <Products.Five.metaclass.WorkflowAction object at 0x...>


Getting the Request
-------------------

This function will return the global request object::

    >>> api.get_request()
    <HTTPRequest, URL=http://nohost>


Getting a Group
---------------

Users in Bika LIMS are managed in groups. A common group is the `Clients` group,
where all users of client contacts are grouped.
This function gives easy access and is also idempotent::

    >>> clients_group = api.get_group("Clients")
    >>> clients_group
    <GroupData at /plone/portal_groupdata/Clients used for /plone/acl_users/source_groups>

    >>> api.get_group(clients_group)
    <GroupData at /plone/portal_groupdata/Clients used for /plone/acl_users/source_groups>

Non-existing groups are not found::

    >>> api.get_group("NonExistingGroup")


Getting a User
--------------

Users can be fetched by their user id. The function is idempotent and handles
user objects as well::

    >>> from plone.app.testing import TEST_USER_ID
    >>> user = api.get_user(TEST_USER_ID)
    >>> user
    <MemberData at /plone/portal_memberdata/test_user_1_ used for /plone/acl_users>

    >>> api.get_user(api.get_user(TEST_USER_ID))
    <MemberData at /plone/portal_memberdata/test_user_1_ used for /plone/acl_users>

Non-existing users are not found::

    >>> api.get_user("NonExistingUser")


Getting User Properties
-----------------------

User properties, like the email or full name, are stored as user properties.
This means that they are not on the user object. This function retrieves these
properties for you::

    >>> properties = api.get_user_properties(TEST_USER_ID)
    >>> sorted(properties.items())
    [('description', ''), ('email', ''), ('error_log_update', 0.0), ('ext_editor', False), ...]

    >>> sorted(api.get_user_properties(user).items())
    [('description', ''), ('email', ''), ('error_log_update', 0.0), ('ext_editor', False), ...]

An empty property dict is returned if no user could be found::

    >>> api.get_user_properties("NonExistingUser")
    {}

    >>> api.get_user_properties(None)
    {}


Getting Users by their Roles
----------------------------

    >>> from operator import methodcaller

Roles in Bika LIMS are basically a name for one or more permissions. For
example, a `LabManager` describes a role which is granted the most permissions.

To see which users are granted a certain role, you can use this function::

    >>> labmanagers = api.get_users_by_roles(["LabManager"])
    >>> sorted(labmanagers, key=methodcaller('getId'))
    [<PloneUser 'test_labmanager'>, <PloneUser 'test_labmanager1'>, <PloneUser 'test-user'>]

A single value can also be passed into this function::

    >>> sorted(api.get_users_by_roles("LabManager"), key=methodcaller('getId'))
    [<PloneUser 'test_labmanager'>, <PloneUser 'test_labmanager1'>, <PloneUser 'test-user'>]


Getting the Current User
------------------------

Getting the current logged in user::

    >>> api.get_current_user()
    <MemberData at /plone/portal_memberdata/test_user_1_ used for /plone/acl_users>
