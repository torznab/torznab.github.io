.. |error-201| replace:: error ``201 - Incorrect parameter``

Introduction
============

Torznab is an api specification based on the Newznab WebAPI. The api is built around a simple xml/rss feed with filtering and paging capabilities.

The Torznab standard strives to be completely compliant with Newznab, insofar it does not conflict with it's primary goal to supply an consistent api for torrents.

|newznab| and |torznab| tags wil be used to designate differences between the two specifications. |br|
The |newznab| tag either designates newznab specific behavior, or functions that are optional in Newznab and not implemented in Torznab. |br|
The |torznab| tag indicates functionality specific to Torznab. Some |torznab| functionality has been implemented later by Newznab, but has not been added to the Newznab WebAPI documentation.

Terminology
===========

This document contains the terms |SHALL|, |MUST|, |SHOULD| and |MAY|. These terms are used in the following meaning:

- |SHALL| and |MUST| means that a particular requirement is mandatory. (The chosen term is dependent on the subject type)
- |SHOULD| means that a particular requirement is recommended but not mandatory.
- |MAY| means that a particular requirement is entirely optional.

Torznab Api Specification
=========================

Formatting
----------

The api uses HTTP/HTTPS calls with xml results using the utf-8 encoding.

Standard Queries
----------------

Generic
~~~~~~~

All api calls in torznab consist of a HTTP GET call to the ``/api`` endpoint with several query parameters.

=================   =======================================================
t={...}             Mandatory. Specifies which function should be invoked.
apikey={...}        Authentication key which may be mandatory. Authenticates the user.
=================   =======================================================

The ``apikey`` may or may not be mandatory depending on the indexer, most newznab indexers require apikey authentication, but some public indexers do not.

**HTTP Request**

``GET /api?t={function}[&apikey={apikey}]``

**HTTP Response**

``200 OK``

Function Overview
~~~~~~~~~~~~~~~~~

============   =======================================================
caps           Returns the capabilities of the api.
search         Free text search query.
tvsearch       Search query with tv specific query params and filtering.
movie          Search query with movie specific query params and filtering.
music          Search query with music specific query params and filtering.
book           Search query with book specific query params and filtering.
details        |newznab| Returns all details about a particular item.
getnfo         |newznab| Returns an nfo for a particular item.
get            |newznab| Returns nzb for the specified item.
card-add       |newznab| Adds item to the users cart.
card-del       |newznab| Removes item from the users cart.
comments       |newznab| Returns all comments known about an item.
comments-add   |newznab| Adds a comment to an item.
register       |newznab| Register a new user account.
user           |newznab| Retrieves information about an user account.
============   =======================================================

Administrative functions
~~~~~~~~~~~~~~~~~~~~~~~~

'caps' function
~~~~~~~~~~~~~~~

The caps query should be supported by all implementations. It specifies the api version and other metadata useful to the client.

|newznab| apikey authentication is not required for this function. |br|
|torznab| apikey authentication is usually not required for this function, but some private trackers require it for security reasons.

**HTTP Request**

``GET /api?t=caps``

**Response**

See :ref:`capabilities`.


Search functions
~~~~~~~~~~~~~~~~

Generic
~~~~~~~

All search functions have certain query params that control the results.

**Optional Parameters**

=================   =======================================================
cat                 Comma separated list of category numbers that should be included.
attrs               Comma separated list of extended attribute names that should be included in the results.
extended=1          Specifies that all extended attributes should be included in the results. Overrules `attrs`.
offset              Number of items to skip in the result.
limit               Number of results to return. Limited by the `limits` element in the :ref:`capabilities`.
=================   =======================================================

If ``offset`` is out of bounds, then an empty result set will be returned.

'search'
~~~~~~~~

The search function provides free text search support without specific movie/tv/etc query parameters.

**HTTP Request**

``GET /api?t=search``

**Optional Parameters**

=================   =======================================================
q                   Free text search query. Indexer specific implementations and patterns may apply.
tag                 |torznab| Comma separated list of tags that must (not) be present in the results. 
=================   =======================================================

**Response**

See 'standard response'.


.. _capabilities:

Capabilities
------------

The capabilities of the newznab / torznab indexer service is retrievable via the ``/api?t=caps`` endpoint. It's intended to provide the client with information about which search capabilities, categories, groups etc the indexer supports.

This information is crucial since it allows the indexer to implement a limited number of search parameters, clients use the `supportedParams` list to determine whether efficient 'indexed' queries are available or that the client must use the generic `q` free text search capability.

The result of the api call is a simple XML document with the following format:

Caps Format
~~~~~~~~~~~

.. example::

   .. code-block:: xml

      <?xml version="1.0" encoding="UTF-8"?>
      <caps>
         <server version="1.1" title="..." strapline="..."
               email="..." url="http://indexer.local/"
               image="http://indexer.local/content/banner.jpg" />
         <limits max="100" default="50" />
         <retention days="400" />
         <registration available="yes" open="yes" />

         <searching>
            <search available="yes" supportedParams="q" />
            <tv-search available="yes" supportedParams="q,rid,tvdbid,season,ep" />
            <movie-search available="no" supportedParams="q,imdbid,genre" />
            <audio-search available="no" supportedParams="q" />
            <book-search available="no" supportedParams="q" />
         </searching>

         <categories>
            <category id="2000" name="Movies">
               <subcat id="2010" name="Foreign" />
            </category>
            <category id="5000" name="TV">
               <subcat id="5040" name="HD" />
               <subcat id="5070" name="Anime" />
            </category>
         </categories>

         <groups>
            <group id="1" name="alt.binaries...." description="..." lastupdate="..." />
         </groups>

         <genres>
            <genre id="1" categoryid="5000" name="Kids" />
         </genres>

         <tags>
            <tag name="anonymous" description="Uploader is anonymous" />
            <tag name="trusted" description="Uploader has high reputation" />
            <tag name="internal" description="Uploader is an internal release group" />
         </tags>
      </caps>

=================   =======================================================
server              Information about the server itself, all attributes are free to be changed by the indexer.
limits              Specifies the maximum and default values for the `limit` query parameter.
retention           |newznab| Usenet retention in days. |br|
                    |torznab| Should be omitted.
searching           Specifies the search capabilities of individual search modes. See :ref:`supported-params`.
categories          List of known categories for use in the `cat` query parameter.
groups              |newznab| Optional list of usenet groups that are indexed. |br|
                    |torznab| No current application specified.
genres              Optional list of known genres for use in the `genre` query parameter.
tags                |torznab| Optional list of supported tags for use in the `tag` query parameter.
=================   =======================================================

.. _supported-params:

Supported Params
~~~~~~~~~~~~~~~~

Extended attributes
===================

Newznab and Torznab use several of the attributes and elements defined by the RSS specification, but also add several additional attributes in a separate xml namespace.

|newznab| All attributes reside in the ``xmlns:newznab="http://www.newznab.com/DTD/2010/feeds/attributes/"`` namespace. |br|
|torznab| All attributes reside in the ``xmlns:torznab="http://torznab.com/schemas/2015/feed"`` namespace.

Extended attributes follow the format:

.. code-block:: xml

   <torznab:attr name="seeders" value="1" />
   <torznab:attr name="category" value="5000" />
   <torznab:attr name="category" value="5040" />

Several extended attributes may occur multiple times, such as `category`.

Clients |SHALL| handle attributes with the same name but with different values as a list instead of a singular value. Clients |SHALL| ignore duplicate values.

Predefined attributes
---------------------

These predefined attributes are optional except for the `size` and `category` attribute.

   .. table:: Predefined newznab & torznab extended attributes
      :class: small

      ====================  ===================  =======  =============
      name                  scope                type     description  
      ====================  ===================  =======  =============
      size                  all                  integer  Size in bytes
      category              all                  integer  Category id. May occur multiple times.
      tag                   all                  string   |torznab| Individual tag. May occur multiple times.
      guid                  all                  string   Unique release guid.
      files                 all                  integer  Number of files in the release
      poster                all                  string   |newznab| NNTP poster (eg. ``yenc@power-poster``) |br| 
                                                          |torznab| Username that uploaded the release to the tracker.
      group                 all                  string   |newznab| NNTP Group(s) (eg. ``a.b.group, a.b.tv``)
      team                  all                  string   Team doing the release. (Release Group name)
      grabs                 all                  integer  Number of times the release was fetched from the indexer.
      seeders               all                  integer  |torznab| Number of active seeders according to the tracker(s).
      leechers              all                  integer  |torznab| Number of active leechers (non-seeders) according to the tracker(s).
      peers                 all                  integer  |torznab| Number of active peers (seeders and leechers) according to the tracker(s).
      infohash              all                  string   |torznab| Torrent infohash as lower-case hexadecimal string.
      magneturl             all                  url      |torznab| Magnet uri.
      seedtype              all                  string   |torznab| Specifies which seed criteria must be met. Possible values are: `ratio`, `seedtime`, `both` or `either` (default).
      minimumratio          all                  decimal  |torznab| Minimum ratio required before the torrent client may stop seeding.
      minimumseedtime       all                  decimal  |torznab| Minimum time in seconds that the torrent must be actively seeded by the client before it may be stopped.
      downloadvolumefactor  all                  decimal  |torznab| Multiplier for the download volume that counts toward the user's account on the tracker. Freeleech would be `0.0`.
      uploadvolumefactor    all                  decimal  |torznab| Multiplier for the upload volume that counts toward the user's account on the tracker.
      --------------------  -------------------  -------  -------------
      password              all                  integer  Whether the archive is passworded (`0`=no, `1`=rar password, `2`=contains inner archive)
      comments              all                  integer  Number of comments
      usenetdate            all                  string   |newznab| Date the release was uploaded to usenet (eg. ``Tue, 22 Jun 2010 06:54:22 +0100``)
      nfo                   all                  integer  Whether an .nfo file is available. (`0` no, `1` yes)
      info                  all                  string   Url for the .nfo file.
      year                  all                  integer  Release year
      --------------------  -------------------  -------  -------------
      coverurl              all                  string   Cover image url
      backdropurl           all                  string   Backdrop image url
      review                all                  string   Critics review
      --------------------  -------------------  -------  -------------
      season                tv                   integer  Season number
      episode               tv                   integer  Episode number
      rageid                tv                   integer  Detected TVRage ID. (TVRage.com, now defunct)
      tvtitle               tv                   string   Human readable Show Title (originally from TV Rage)
      tvairdate             tv                   string   Air date for the episode (eg. ``Tue, 22 Jun 2010 06:54:22 +0100``)
      tvdbid                tv                   integer  TheTVDB show id associated with this release.
      tvmazeid              tv                   integer  TVMaze show id associated with this release.
      --------------------  -------------------  -------  -------------
      genre                 tv, movies           string   Genre (eg. ``Horror, Comedy``)   
      video                 tv, movies           string   Video Codec (eg. ``x264``)
      audio                 tv, movies, audio    string   Audio Codec (eg. ``AC3 2.0 @ 384 kbs``)
      resolution            tv, movies           string   Video resolution (eg. ``1280x716 1.78:1``)
      framerate             tv, movies           string   Video framerate (eg. ``23.976 fps``)
      language              tv, movies, audio    string   Video/Audio languages (eg. ``English``)
      subs                  tv, movies           string   Subtitle languages (eg. ``English, Spanish``)
      --------------------  -------------------  -------  -------------
      imdb                  movies               string   Imdb ID. (www.imdb.com)
      imdbscore             movies               string   Imdb score (eg. ``5/10``)
      imdbtitle             movies               string   Imdb title
      imdbtagline           movies               string   Imdb tagline
      imdbplot              movies               string   Imdb plot
      imdbyear              movies               integer  Imdb year
      imdbdirector          movies               string   Imdb director
      imdbactors            movies               string   Imdb actors
      --------------------  -------------------  -------  -------------
      artist                music                string   Artist name
      album                 music                string   Album name
      publisher             music                string   Publisher name
      tracks                music                string   Track listing (eg. ``Track 1|Track 2|Track 3``)
      --------------------  -------------------  -------  -------------
      booktitle             book                 string   Book title
      publishdate           book                 string   Date the book was published
      author                book                 string   Booth author
      pages                 book                 integer  Number of book pages
      ====================  ===================  =======  =============

Client Implementation Guidelines
================================

.. TODO:: Needs to be implemented.

Service Implementation Guidelines
=================================

These guidelines aim to eliminate ambiguity in the expected behavior of the service implementation. It also describes common pitfalls for service implementations.
Service implementations may deviate from these guidelines and as such clients should not assume services to conform.

Parameters
----------

The query parameter's name |MUST| be evaluated case-insensitive.

The service |SHALL| verify for each parameter whether the value is within the accepted range or, unless specified, return |error-201|.

'cat' parameter
~~~~~~~~~~~~~~~

The `cat` parameter is a comma separated list of categories for which the client wishes to receive the results.
The service |SHOULD| fully implement this query parameter.

The `cat` parameter |MUST| be verified to contain a comma separated list of integer numbers. For example using the :regexp:`^\d+(,\d+)*$` regular expression. If the parameter is invalid, then the |error-201| |MUST| be returned.

If the `cat` parameter is not included in the query sent by the client, the service |SHOULD| return the results for all categories.

Unknown categories |MUST| be silently ignored, if all categories specified by the `cat` parameter are unknown then the service |SHALL| return an empty result set.

The service |SHALL| return an item only once if it exists in multiple specified categories.

.. example::
    ``cat=4030,5070,1234`` returns results that exist in category 4030 and/or 5070. 1234, being an unknown category, is ignored.

    ``cat=1234`` returns no results at all since there are no items in the specified unknown category.

'attrs' and 'extended' parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The `attrs` parameter is a comma separated list of extended attribute names, allowing the client to request that specific extended attributes are included in the results.
The `extended` parameter is used to request *all* supported extended attributes.

The service |SHALL| either implement both parameters, only the `extended` parameter or none.

If the service does not implement the query parameters, then all supported extended attributes |MUST| be included in the result set.

The service |SHALL| accept the `extended` parameter value `1` and `0` to determine whether all extended attributes are requested by the client.
The service |MAY| also accept the values `true`/`yes` and `false`/`no`, with any capitalization. Any other value |SHOULD| result in |error-201|.

If the `extended` parameter is not 'true' then the service |SHALL| include all supported extended attribute specified by the `attrs` parameter.

The `attrs` parameter must be validated against invalid characters. For example using the :regexp:`^[a-zA-Z]+(,[a-zA-Z]+)*$` regular expression.
Unknown extended attributes specified in the `attrs` parameter |MUST| be silently ignored.

'offset' and 'limit' parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The `offset` and `limit` parameters is used to specify which portion of the result set should be returned to the client. The service |SHALL| implement both parameters.

The service |SHALL| verify whether both values are integers greater or equal to zero. Otherwise the |error-201| |MUST| be returned.

The `offset` specifies how many items in the result set should be skipped. The offset |MUST| default to the value zero.

The service |SHALL| return an empty result if offset is equal or exceeds the total number of results.

The `limit` parameter specifies how many results should be returned.
The default and maximum value for `limit` is specified by the :ref:`capabilities`. The service |SHOULD| automatically limit the value to the maximum.

'tag' parameter
~~~~~~~~~~~~~~~

Added to |torznab| in version 1.3.

The service |MAY| implement the `tag` parameter, if it does then the `supportedParams` attribute in the :ref:`capabilities` |MUST| include 'tag'.

The `tag` parameter is a comma separated list of tags that must (not) be present in the results. A prefixed hyphen (`-`) means exclusion. The order of tags in the list is irrelevant.

Returned results |MUST| match all tags specified by the parameter.

The service |SHALL| treat unknown tags as if no release has the tag and behave appropriatly based on whether it's an inclusion or exclusion.

A list of supported tags |SHOULD| be included in the :ref:`capabilities`.

.. example:: ``tag=anonymous,-remake`` indicates that all releases must have the tag 'anonymous' and must not have the tag 'remake'.


.. TODO:: Incomplete


General api
-----------

.. TODO:: Needs to be implemented.