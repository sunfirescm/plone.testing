Zope Publisher layers
---------------------

The Zope Publisher layers are found in the module ``plone.testing.publisher``::

    >>> from plone.testing import publisher

For testing, we need a testrunner:::

    >>> from zope.testrunner import runner

ZCML directives
~~~~~~~~~~~~~~~

The ``publisher.PUBLISHER_DIRECTIVES`` layer extends the ``zca.ZCML_DIRECTIVES`` layer to extend its ZCML configuration context with the ``zope.app.publisher`` and ``zope.security`` directives available.
It also extends ``security.CHECKERS``.::

    >>> from plone.testing import zca, security

    >>> "%s.%s" % (publisher.PUBLISHER_DIRECTIVES.__module__, publisher.PUBLISHER_DIRECTIVES.__name__,)
    'plone.testing.publisher.PublisherDirectives'

    >>> publisher.PUBLISHER_DIRECTIVES.__bases__
    (<Layer 'plone.testing.zca.ZCMLDirectives'>, <Layer 'plone.testing.security.Checkers'>)

Before the test, we cannot use e.g.
the ``<permission />`` or ``<browser:view />`` directives without loading the necessary ``meta.zcml`` files.::

    >>> from zope.configuration import xmlconfig
    >>> from zope.configuration.exceptions import ConfigurationError
    >>> try:
    ...     xmlconfig.string("""\
    ...     <configure package="plone.testing"
    ...         xmlns="http://namespaces.zope.org/zope"
    ...         xmlns:browser="http://namespaces.zope.org/browser"
    ...         i18n_domain="plone.testing.tests">
    ...         <permission id="plone.testing.Test" title="plone.testing: Test" />
    ...         <browser:view
    ...             for="*"
    ...             name="plone.testing-test"
    ...             class="plone.testing.tests.DummyView"
    ...             permission="zope.Public"
    ...             />
    ...     </configure>""")
    ... except ConfigurationError as e:
    ...     True
    True

Layer setup creates a configuration context we can use to load further configuration.::

    >>> options = runner.get_options([], [])
    >>> setupLayers = {}
    >>> runner.setup_layer(options, publisher.PUBLISHER_DIRECTIVES, setupLayers)
    Set up plone.testing.zca.LayerCleanup in ... seconds.
    Set up plone.testing.zca.ZCMLDirectives in ... seconds.
    Set up plone.testing.security.Checkers in ... seconds.
    Set up plone.testing.publisher.PublisherDirectives in ... seconds.


Let's now simulate a test that uses this configuration context to load the same ZCML string.::

    >>> zca.ZCML_DIRECTIVES.testSetUp()
    >>> security.CHECKERS.testSetUp()
    >>> publisher.PUBLISHER_DIRECTIVES.testSetUp()

    >>> context = zca.ZCML_DIRECTIVES['configurationContext'] # would normally be self.layer['configurationContext']
    >>> xmlconfig.string("""\
    ... <configure package="plone.testing"
    ...     xmlns="http://namespaces.zope.org/zope"
    ...     xmlns:browser="http://namespaces.zope.org/browser"
    ...     i18n_domain="plone.testing.tests">
    ...     <permission id="plone.testing.Test" title="plone.testing: Test" />
    ...     <browser:view
    ...         for="*"
    ...         name="plone.testing-test"
    ...         class="plone.testing.tests.DummyView"
    ...         permission="zope.Public"
    ...         />
    ... </configure>""", context=context) is context
    True

The permission and view are now registered:::

    >>> from zope.component import queryUtility
    >>> from zope.security.interfaces import IPermission

    >>> queryUtility(IPermission, name=u"plone.testing.Test")
    <zope.security.permission.Permission object at ...>

    >>> from zope.interface import Interface
    >>> from zope.publisher.interfaces.browser import IDefaultBrowserLayer
    >>> from zope.component import getSiteManager
    >>> siteManager = getSiteManager()

    >>> [x.factory for x in siteManager.registeredAdapters()
    ...  if x.provided==Interface and x.required==(Interface, IDefaultBrowserLayer)
    ...   and x.name==u"plone.testing-test"]
    [<class '....plone.testing-test'>]

We can then simulate test tear-down:::

    >>> publisher.PUBLISHER_DIRECTIVES.testTearDown()
    >>> security.CHECKERS.testTearDown()
    >>> zca.ZCML_DIRECTIVES.testTearDown()

Note that you'd normally combine this layer with the ``zca.UNIT_TESTING`` or a similar layer to automatically tear down the component architecture between each test.
Here, we need to do it manually.::

    >>> from zope.component.testing import tearDown
    >>> tearDown()

Layer tear-down does nothing.::

    >>> runner.tear_down_unneeded(options, [], setupLayers)
    Tear down plone.testing.publisher.PublisherDirectives in ... seconds.
    Tear down plone.testing.zca.ZCMLDirectives in ... seconds.
    Tear down plone.testing.zca.LayerCleanup in ... seconds.
    Tear down plone.testing.security.Checkers in ... seconds.

    >>> zca.ZCML_DIRECTIVES.get('configurationContext', None) is None
    True
