.. _sec-api-util:

****
Util
****

.. _sec-api-util-test:

Various tests
=============

.. http:post:: /api/util/test

   Provides commands to test paths or URLs for correctness.

   Used by OctoPrint to validate paths or URLs that the user needs to enter in the
   settings.

   The following commands are supported at the moment:

   .. _sec-api-util-test-path:

   path
     Tests whether or provided path exists and optionally if it also is either a file
     or a directory and whether OctoPrint's user has read, write and/or execute permissions
     on it. Supported parameters are:

       * ``path``: The file system path to test. Mandatory.
       * ``check_type``: ``file`` or ``dir`` if the path should not only be checked for
         existence but also whether it is of the specified type. Optional.
       * ``check_access``: A list of any of ``r``, ``w`` and ``x``. If present it will also
         be checked if OctoPrint has read, write, execute permissions on the specified path.
       * ``allow_create_dir``: If ``check_type`` is provided and set to ``dir``, this will allow
         OctoPrint to create the target ``path`` as a directory if it doesn't yet exist to allow
         for further tests.
       * ``check_writable_dir``: If ``check_type`` is provided and set to ``dir``, this will
         check that the provided ``path`` is a writable directory. OctoPrint not only check if the
         permissions on the directory allow for writing but also attempt to write (and delete) a
         small test file ``.testballoon.txt`` to the directory to test if writing is actually
         possible.

     The ``path`` command returns a :http:statuscode:`200` with a :ref:`path test result <sec-api-util-datamodel-pathtestresult>`
     when the test could be performed. The status code of the response does NOT reflect the
     test result!

   .. _sec-api-util-test-url:

   url
     Tests whether a provided URL responds. Request method and expected status codes can
     optionally be specified as well. Supported parameters are:

       * ``url``: The URL to test. Mandatory.
       * ``method``: The request method to use for the test. Optional, defaults to ``HEAD``.
       * ``timeout``: A timeout for the request, in seconds. If no reply from the tested URL has been
         received within this time frame, the check will be considered a failure. Optional, defaults to 3 seconds.
       * ``validSsl``: Whether to validate the SSL connection if the ``url`` happens to be an HTTPS URL or not. Optional,
         defaults to ``True``.
       * ``basicAuth``: A dictionary with the keys ``username`` and ``password`` to use for basic authentication. Optional.
       * ``digestAuth``: A dictionary with the keys ``username`` and ``password`` to use for digest authentication. Optional.
       * ``bearerAuth``: A string with the bearer token to use for bearer authentication. Optional.
       * ``status``: The status code(s) or named status range(s) to test for. Can be either a single
         value or a list of either HTTP status codes or any of the following named status ranges:

           * ``informational``: Status codes from 100 to 199
           * ``success``: Status codes from 200 to 299
           * ``redirection``: Status codes from 300 to 399
           * ``client_error``: Status codes from 400 to 499
           * ``server_error``: Status codes from 500 to 599
           * ``normal``: Status codes from 100 to 399
           * ``error``: Status codes from 400 to 599
           * ``any``: Any status code starting from 100

         The test will past the status code check if the status returned by the URL is within any of
         the specified ranges.
       * ``response``: If set to either ``true``, ``json`` or ``bytes``, the response body and the response headers
         from the URL check will be returned as part of the check result as well. ``json`` will attempt
         to parse the response as json and return the parsed result. ``true`` or ``bytes`` will base64 encode the body
         and return that.
       * ``content_type_whitelist``: Optional array of supported content types. If set and the URL returns a content
         type not included in this list, the test will fail. E.g. ``["image/*", "text/plain"]``.
       * ``content_type_blacklist``: Optional array of unsupported content types. If set and the URL returns a content
         type included in this list, the test will fail. E.g. ``["video/*"]``. Can be used together with ``content_type_whitelist``
         to further limit broader content type definition, e.g. by putting ``image/*`` into the whitelist, but disallowing
         PNG by including ``image/png`` on the blacklist.

     The ``url`` command returns :http:statuscode:`200` with a :ref:`URL test result <sec-api-util-datamodel-urltestresult>`
     when the test could be performed. The status code of the response does NOT reflect the
     test result!

   .. _sec-api-util-test-server:

   server
     Tests whether a provided server identified by host and port can be reached. Protocol can optionally be specified
     as well. Supported parameters are:

       * ``host``: The host to test. IP or host name. Mandatory.
       * ``port``: The port to test. Integer. Mandatory.
       * ``protocol``: The protocol to test with. ``tcp`` or ``udp``. Optional, defaults to ``tcp``.
       * ``timeout``: A timeout for the test, in seconds. If no successful connection to the server could be established
         within this time frame, the check will be considered a failure. Optional, defaults to 3.05 seconds.

     The ``server`` command returns :http:statuscode:`200` with a :ref:`Server test result <sec-api-util-datamodel-servertestresult>`
     when the test could be performed. The status code of the response does NOT reflect the test result!

   resolution
     Tests whether a provided hostname can be resolved (via DNS lookup). Supported parameters are:

       * ``name``: The host name to test. Mandatory.

     The ``resolution`` command returns :http:statuscode:`200` with a :ref:`Resolution test result <sec-api-util-datamodel-resolutiontestresult>`
     when the test could be performed. The status code of the response does NOT reflect the test result!

   address
     Tests whether a provided address (or, if none is provided, the client's remote address) is
     a LAN address and if so returns the subnet specifier in CIDR format.

       * ``address``: the address to test. If not set, the client's remote address will be used

     The ``address`` command return :http:statuscode:`200` with a :ref:`Address test result <sec-api-util-datamodel-addresstestresult>`
     when the test could be performed. The status code of the response does NOT reflect the test result!

   Requires the ``ADMIN`` permission.

   **Example 1**

   Test whether a path exists and is a file readable and executable by OctoPrint.

   .. sourcecode:: http

      POST /api/util/test HTTP/1.1
      Host: example.com
      X-Api-Key: abcdef...
      Content-Type: application/json

      {
        "command": "path",
        "path": "/some/path/to/a/file",
        "check_type": "file",
        "check_access": ["r", "x"]
      }

   .. sourcecode:: HTTP

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "path": "/some/path/to/a/file",
        "exists": true,
        "typeok": true,
        "access": true,
        "result": true
      }

   **Example 2**

   Test whether a path exists which doesn't exist.

   .. sourcecode:: http

      POST /api/util/test HTTP/1.1
      Host: example.com
      X-Api-Key: abcdef...
      Content-Type: application/json

      {
        "command": "path",
        "path": "/some/path/to/a/missing_file",
        "check_type": "file",
        "check_access": ["r", "x"]
      }

   .. sourcecode:: HTTP

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "path": "/some/path/to/a/missing_file",
        "exists": false,
        "typeok": false,
        "access": false,
        "result": false
      }

   **Example 3**

   Test whether a path exists and is a file which is a directory.

   .. sourcecode:: http

      POST /api/util/test HTTP/1.1
      Host: example.com
      X-Api-Key: abcdef...
      Content-Type: application/json

      {
        "command": "path",
        "path": "/some/path/to/a/folder",
        "check_type": "file"
      }

   .. sourcecode:: HTTP

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "path": "/some/path/to/a/folder",
        "exists": true,
        "typeok": false,
        "access": true,
        "result": false
      }

   **Example 4**

   Test whether a URL returns a normal status code for a HEAD request.

   .. sourcecode:: http

      POST /api/util/test HTTP/1.1
      Host: example.com
      X-Api-Key: abcdef...
      Content-Type: application/json

      {
        "command": "url",
        "url": "http://example.com/some/url"
      }

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "url": "http://example.com/some/url",
        "status": 200,
        "result": true
      }

   **Example 5**

   Test whether a URL can be called at all via GET request, provide its raw body. Set a timeout of 1s.

   .. sourcecode:: http

      POST /api/util/test HTTP/1.1
      Host: example.com
      X-Api-Key: abcdef...
      Content-Type: application/json

      {
        "command": "url",
        "url": "http://example.com/some/url",
        "method": "GET",
        "timeout": 1.0,
        "status": "any",
        "response": true
      }

   .. sourcecode:: HTTP

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "url": "http://example.com/some/url",
        "status": 200,
        "result": true,
        "response": {
          "headers": {
            "content-type": "image/gif"
          },
          "content": "R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7"
        }
      }

   **Example 6**

   Test whether a server is reachable on a given port via TCP.

   .. sourcecode:: http

      POST /api/util/test HTTP/1.1
      Host: example.com
      X-Api-Key: abcdef...
      Content-Type: application/json

      {
        "command": "server",
        "host": "8.8.8.8",
        "port": 53
      }

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "host": "8.8.8.8",
        "port": 53,
        "protocol": "tcp",
        "result": true
      }

   **Example 7**

   Test whether a host name can be resolved successfully.

   .. sourcecode:: http

      POST /api/util/test HTTP/1.1
      Host: example.com
      X-Api-Key: abcdef...
      Content-Type: application/json

      {
        "command": "resolution",
        "name": "octoprint"
      }

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "name": "octoprint.org",
        "result": true
      }

   **Example 8**

   Test whether the client's address is a LAN address.

   .. sourcecode:: http

      POST /api/util/test HTTP/1.1
      Host: example.com
      X-Api-Key: abcdef...
      Content-Type: application/json

      {
        "command": "address"
      }

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "address": "192.168.1.3",
        "is_lan_address": true,
        "subnet": "192.168.0.0/16"
      }

   **Example 9**

   Test whether ``8.8.8.8`` is a LAN address.

   .. sourcecode:: http

      POST /api/util/test HTTP/1.1
      Host: example.com
      X-Api-Key: abcdef...
      Content-Type: application/json

      {
        "command": "address",
        "address": "8.8.8.8"
      }

   .. sourcecode:: http

      HTTP/1.1 200 OK
      Content-Type: application/json

      {
        "address": "8.8.8.8",
        "is_lan_address": false
      }

   :json command:            The command to execute, currently either ``path`` or ``url``
   :json path:               ``path`` command only: the path to test
   :json check_type:         ``path`` command only: the type of path to test for, either ``file`` or ``dir``
   :json check_access:       ``path`` command only: a list of access permissions to check for
   :json allow_create_dir:   ``path`` command and ``checktype`` of ``dir`` only: whether to allow creation of the
                             directory if it doesn't yet exist (``true``) or not (``false``, default)
   :json check_writable_dir: ``path`` command and ``checktype`` of ``dir`` only: whether to test if the directory
                             is writable by also trying to create a test file in it (``true``) or not (``false``, default)
   :json url:                ``url`` command only: the URL to test
   :json status:             ``url`` command only: one or more expected status codes
   :json method:             ``url`` command only: the HTTP method to use for the check
   :json timeout:            ``url`` and ``server`` commands only: the timeout for the test request
   :json response:           ``url`` command only: whether to include response data and if so in what form
   :json host:               ``server`` command only: the server to test
   :json port:               ``server`` command only: the port to test
   :json protocol:           ``server`` command only: the protocol to test
   :json name:               ``resolution`` command only: the host name to test
   :json address:            ``address`` command only: the address to test
   :statuscode 200:          No error occurred

.. _sec-api-util-datamodel:

Data model
==========

.. _sec-api-util-datamodel-pathtestresult:

Path test result
----------------

.. list-table::
   :widths: 15 5 10 30
   :header-rows: 1

   * - Name
     - Multiplicity
     - Type
     - Description
   * - ``path``
     - 1
     - string
     - The path that was tested.
   * - ``exists``
     - 1
     - bool
     - ``true`` if the path exists, ``false`` otherwise.
   * - ``typeok``
     - 1
     - bool
     - ``true`` if a type check was not requested or it passed, ``false`` otherwise
   * - ``access``
     - 1
     - bool
     - ``true`` if a permission check was not requested or it passed, ``false`` otherwise
   * - ``result``
     - 1
     - bool
     - ``true`` if the overall check passed, ``false`` otherwise

.. _sec-api-util-datamodel-urltestresult:

URL test result
---------------

.. list-table::
   :widths: 15 5 10 30
   :header-rows: 1

   * - Name
     - Multiplicity
     - Type
     - Description
   * - ``url``
     - 1
     - string
     - The URL that was tested.
   * - ``status``
     - 1
     - int
     - The status code returned by the URL, 0 in case of a timeout.
   * - ``result``
     - 1
     - bool
     - ``true`` if the check passed.
   * - ``response``
     - 0..1
     - string or object
     - If ``response`` in the request was set to ``bytes``: The base64 encoded body of the checked URL's response.
       If ``response`` in the request was set to ``json``: The json decoded body of the checked URL's response.
       Not present if ``response`` in the request was not set.
   * - ``headers``
     - 0..1
     - object
     - A dictionary with all headers of the checked URL's response. Only present if ``response`` in the
       request was set.

.. _sec-api-util-datamodel-servertestresult:

Server test result
------------------

.. list-table::
   :widths: 15 5 10 30
   :header-rows: 1

   * - Name
     - Multiplicity
     - Type
     - Description
   * - ``host``
     - 1
     - string
     - The host that was tested.
   * - ``port``
     - 1
     - int
     - The port that was tested
   * - ``protocol``
     - 1
     - string
     - The protocol that was tested, ``tcp`` or ``udp``
   * - ``result``
     - 1
     - bool
     - ``true`` if the check passed.

.. _sec-api-util-datamodel-resolutiontestresult:

Resolution test result
----------------------

.. list-table::
   :widths: 15 5 10 30
   :header-rows: 1

   * - Name
     - Multiplicity
     - Type
     - Description
   * - ``name``
     - 1
     - string
     - The host name that was tested.
   * - ``result``
     - 1
     - bool
     - ``true`` if the check passed.

.. _sec-api-util-datamodel-addresstestresult:

Address test result
-------------------

.. list-table::
   :widths: 15 5 10 30
   :header-rows: 1

   * - Name
     - Multiplicity
     - Type
     - Description
   * - ``address``
     - 1
     - string
     - The address that was tested.
   * - ``is_lan_address``
     - 1
     - bool
     - ``true`` if the address is a LAN address, false otherwise.
   * - ``subnet``
     - 0..1
     - string
     - The detected subnet, if address is a LAN address.
