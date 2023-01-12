.. role:: python(code)
   :language: python

==
h1
==

.. image:: https://img.shields.io/pypi/pyversions/h1.svg
.. image:: https://img.shields.io/pypi/v/h1.svg
    :target: https://pypi.python.org/pypi/h1


A HackerOne API client for Python. The API closely maps to the REST API that HackerOne provides.
Documentation for their API is `available here <https://api.hackerone.com/docs/v1>`_.

=======
License
=======

MIT

============
Installation
============

For installation via pip:

.. code-block:: bash

    pip install h1

For development, In the project root run:

.. code-block:: bash

    virtualenv env
    source env/bin/activate
    make bootstrap

The manual approach should work as well:

.. code-block:: bash

    python setup.py install
    

for requirements.txt use: 

.. code-block:: python

git+https://github.com/eliuha/h1-python

========
Examples
========

-----------------------
Initializing the Client
-----------------------

.. code-block:: python

    from h1.client import HackerOneClient
    from h1.models import Report
    c = HackerOneClient("YOUR-API-TOKEN-IDENTIFIER", "YOUR-API-TOKEN")

-------------------------------------------
Getting all reports created in the last day
-------------------------------------------

:python:`HackerOneClient.find_resources()` allows you to specify a resource to find (only :python:`Report` is
supported for now) and some criteria to filter on. The only *required* filter is :python:`program`, which
must be set to the target HackerOne program's name. Any additional filters may be passed as kwargs,
and everything in `HackerOne's filter documentation <https://api.hackerone.com/docs/v1#/reports/query>`_
should be supported.

For example, here's how we'd get all reports created in the past 24 hours:

.. code-block:: python

    import datetime as dt
    day_ago = dt.datetime.now() - dt.timedelta(days=1)
    listing = c.find_resources(Report, program=["test-program"], created_at__gt=day_ago)
    len(listing)
    # 3
    listing[0].title
    # u'This is a test report!'

-----------------------------------------
Getting all resolved reports in a program
-----------------------------------------

Similarly, if we filter on :python:`state` we can get all the :python:`resolved` reports:

.. code-block:: python

    resolved_listing = c.find_resources(Report, program=["test-program"], state=["resolved"])
    resolved_listing[0].title

-------------------------------
Getting a specific report by ID
-------------------------------

:python:`HackerOneClient.get_resource()` allows you to pass a resource type (again, currently just :python:`Report`,)
and an ID to fetch:

.. code-block:: python

    report = c.get_resource(Report, 110306)
    report.title
    # u'Test RCE SQLi'
    report.state
    # u'not-applicable'

------------------------------
Tallying report counts by user
------------------------------

Here's an example of using the client to figure out who your most prolific reporters are:

.. code-block:: python

    from collections import Counter
    reporter_count = Counter()
    all_reports = c.find_resources(Report, program=["test-program"])
    for report in all_reports:
         reporter_count[report.reporter] += 1
    
    print(reporter_count)
    Counter({<User - bestreporter>: 21, <User - another_reporter>: 12, <User - r3p0rt3r>: 2, <User - newbie>: 1})
    
--------------------------
Create a csv
--------------------------


.. code-block:: python

   from h1.client import HackerOneClient
   from h1.models import Report
   from key import h1_token_identifier, h1_api_token

   import datetime as dt


   week_ago = dt.datetime.now() - dt.timedelta(days=600)
   day_ago = dt.datetime.now() - dt.timedelta(days=1)

   c = HackerOneClient(h1_token_identifier, h1_api_token)
   c.s.verify = False # disable SSL checks if you have annoying proxy 

   listing = c.find_resources(Report, program=["program_name"], created_at__gt=week_ago, created_at__lt=day_ago)
   print(len(listing))



   with open("report.csv",'w') as f:
       for item in listing:
           id = item.id
           title = item.title
           weakness = "Undetermined"
           time_to_first_response = item.time_to_first_response.seconds / 3600
           time_to_closed = 'NaN'
           if item.time_to_closed:
               time_to_closed = item.time_to_closed.seconds / 3600
           link = item.html_url

           if item.weakness:
               weakness = item.weakness.name

           line = f"{id},{weakness},{title},{time_to_first_response},{time_to_closed},{link}\n"
           f.write(line)




=============
Running Tests
=============

.. code-block:: bash

    virtualenv env
    source env/bin/activate
    make bootstrap
    make test

