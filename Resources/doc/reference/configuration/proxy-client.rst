proxy_client
============

The proxy client sends invalidation requests to your caching proxy. The
:doc:`Cache Manager </reference/cache-manager>` wraps the proxy client and is
the usual entry point for application interaction with the caching proxy.

You need to configure a client or define your own service for the cache manager
to work.

The proxy client is also directly available as a service
(``fos_http_cache.proxy_client.default`` and ``fos_http_cache.proxy_client.varnish``
, ``fos_http_cache.proxy_client.nginx`` or ``fos_http_cache.proxy_client.symfony``)
that you can use directly.

If you need to adjust the proxy client, you can also configure the ``CacheManager``
with a :ref:`custom proxy client <custom_proxy_client>` that you defined as a
service. In that case, you do not need to configure anything in the
``proxy_client`` configuration section.

varnish
-------

.. code-block:: yaml

    # app/config/config.yml
    fos_http_cache:
        proxy_client:
            varnish:
                tags_header: My-Cache-Tags
                header_length: 1234
                default_ban_headers:
                    Foo: Bar
                http:
                    servers:
                        - 123.123.123.1:6060
                        - 123.123.123.2
                    base_url: yourwebsite.com

``tags_header``
"""""""""""""""

**type**: ``string`` **default**: ``X-Cache-Tags``

Header for sending tag invalidation requests to Varnish.

``header_length``
"""""""""""""""""

**type**: ``integer`` **default**: ``7500``

Maximum header length when invalidating tags. If there are more tags to
invalidate than fit into the header, the invalidation request is split into
multiple requests.

``default_ban_headers``
"""""""""""""""""""""""

**type**: ``array``

Map of header name header value that have to be set on each ban request. This
list is merged with the built-in headers for bans.

``http.servers``
""""""""""""""""

**type**: ``array``

Comma-separated list of IP addresses or host names of your
caching proxy servers. The port those servers will be contacted
defaults to 80; you can specify a different port with ``:<port>``.

When using a multi-server setup, make sure to include **all** proxy servers in
this list. Invalidation must happen on all systems or you will end up with
inconsistent caches.

``http.base_url``
"""""""""""""""""

**type**: ``string``

The hostname (or base URL) where users access your web application. The base
URL may contain a path. If you access your web application on a port other than
80, include that port:

.. code-block:: yaml

    # app/config/config.yml
    fos_http_cache:
        proxy_client:
            varnish:
                http:
                    base_url: yourwebsite.com:8000

.. warning::

    Double-check ``base_url``, for if it is mistyped, no content will be
    invalidated.

See the :ref:`FOSHttpCache library docs <foshttpcache:varnish configuration>`
on how to configure Varnish.

nginx
-----

.. code-block:: yaml

    # app/config/config.yml
    fos_http_cache:
        proxy_client:
            nginx:
                purge_location: /purge
                http:
                    servers:
                        - 123.123.123.1:6060
                        - 123.123.123.2
                    base_url: yourwebsite.com

For ``servers`` and ``base_url``, see above.

``purge_location``
""""""""""""""""""

**type**: ``string``

Separate location that purge requests will be sent to.

See the :ref:`FOSHttpCache library docs <foshttpcache:nginx configuration>`
on how to configure Nginx.

symfony
-------

.. code-block:: yaml

    # app/config/config.yml
    fos_http_cache:
        proxy_client:
            symfony:
                http:
                    servers:
                        - 123.123.123.1:6060
                        - 123.123.123.2
                    base_url: yourwebsite.com

For ``servers`` and ``base_url``, see above.

.. _configuration_noop_proxy_client:

noop
----

.. code-block:: yaml

    # app/config/config_test.yml
    fos_http_cache:
        proxy_client:
            default: noop
            noop: ~

This proxy client supports all invalidation methods, but implements doing
nothing (hence the name "no operation" client). This can be useful for testing.

default
-------

**type**: ``enum`` **options**: ``varnish``, ``nginx``, ``symfony``, ``noop``

.. code-block:: yaml

    # app/config/config.yml
    fos_http_cache:
        proxy_client:
            default: varnish

If there is only one proxy client, it is automatically the default. Only
configure this if you configured more than one proxy client.

The default proxy client that will be used by the cache manager. You can
*configure Nginx, Varnish and Symfony proxy clients in parallel*. There is
however only one cache manager and it will only use the default client.

.. _custom HTTP client:

Custom HTTP Client
------------------

The proxy client uses a ``Http\Client\Utils\HttpMethodsClient`` wrapping a
``Http\Client\HttpClient`` instance. If you need to customize the requests, for
example to send a basic authentication header with each request, you can
configure a service for the ``HttpClient`` and specify that in the
``http_client`` option of any of the cache proxy clients.

Caching Proxy Configuration
---------------------------

You need to configure your caching proxy (Varnish or Nginx) to work with this
bundle. Please refer to the :ref:`FOSHttpCache library’s documentation <foshttpcache:proxy-configuration>`
for more information.
