Title: ErfGeoViewer - Technical Plan
Author: Jason David Yergeau
Base Header Level: 1

# ErfGeoviewer Technical Specification

Introduction
-----------------

The purpose of the ErfGeoviewer is to place cultural heritage collections on a map. It can be used to present a predefined set of items -- for example, all of the windmills in the Netherlands -- or it can be an entry point to explore vast collections across multiple databases by time period, location or semantically by keyword. This document describes the tool's features and a technical approach for building them.

Principles
--------------

- The viewer must be easy to embed into any website. Large websites such as innl.nl will feature the tool prominently on their website, but it should be just as easy to embed these maps by casual bloggers or journalists who want to share their findings.
- The viewer must work for different kinds of collections. It is not possible to design for every variation, but a modular structure will help facilitate contributions to support additional layouts, filters and data types.
- The viewer will use only frontend, open source components. It is written in JavaScript. It will fetch data using an API.
- All code will be open source, and we will encourage community momentum by combining efforts with the Data Atlas.




Document History
-------------------------

- 1.0 (23 June 2015) - Convert the proposal into a technical specification.


----

# Section 1: Product definition

This section describes the features and behaviors of the ErfGeoviewer.

Feature list 
----------------

### F1. Mapping core

This feature includes the implementation of Leaflet.js, which contains the following key features:

- Zoom and drag controls
- Support for keyboard
- Support for touch devices
- Modular architecture
- Support for multiple layers
	- Base tile layer
	- Additional tile layers
	- GeoJSON features layer
	- Image overlays

### F2. Markers

Geocoded objects from a result set (that is, records that contain a longitude and latitude point) can be added to a map in the form of a marker. The markers support the following features:

- Tooltips when hovering over a marker
- Handling of click events to trigger a pop-up 
- Clustering using the Leaflet "markercluster" plugin
- Mapping of a result set to a particular icon

Icons are taken from open source [Maki icon set](https://github.com/mapbox/maki), and implemented using extensions to Leaflet provided by [Mapbox.js](https://github.com/mapbox/mapbox.js/), also open source.

Markers are styled according to the [simplestyle specification, 1.1.0](https://github.com/mapbox/simplestyle-spec/tree/master/1.1.0). This provides the means to modify a marker's size, symbol, color, text and description. 

### F3. Areas

Geocoded objects from a result set that contain geometries such as polygons and lines can also be drawn on the map. These shapes have less interactivity than markers, but can be styled using the [simplestyle specification, 1.1.0](https://github.com/mapbox/simplestyle-spec/tree/master/1.1.0). This provides the means to modify a polygon's stroke (the color and thickness of the border) and its fill color.

### F4. Detail view within the map

The detail view appears after clicking a single object, such as a marker. The fields from that object are then shown. 

The site implementing the ErfGeoviewer can customize this pane by adjusting the default template. By default, the following fields will be displayed if available:

- Title
- Description
- Geographic point (longitude, latitude)
- Image
- Link to external page
- Link to online store

It can be easily extended to show additional fields, such as additional text, images or videos. This is not provided in an automated way by the ErfGeoviewer, but the templating system provides a developer the means to easily adjust what is displayed in the detail view.

### F5. Legend with customizable icons

This feature is closely related to F2 (Markers). A legend can be positioned in any corner of the map (top left, top right, bottom left, or bottom right), and is used to explain the mapping of marker icons with the value of a particular field. 

The legend is configured through a single text field containing tokens. Tokens are converted before being added to the map. 

- Icons from the [Maki icon set](https://github.com/mapbox/maki) can referenced using [maki-ICONNAME] 

### F6. Configuration interface

Some aspects of the map can be configured using a web form. Further customization such is possible by extending the ErfGeoviewer, modifying its templates or CSS, adjusting its default layout, or interacting with the Leaflet API.

See also the section "ErfGeoviewer configuration". 

### F7. Configuration specification

A JSON object will contain the information needed to fetch results, filter and style them on the map into layers.

- Title and description of map
- Collection reference
- Query used to collect records (format of this is not yet known, since the Zoek en Vind API has not been designed)
- Default state frontend filters (reduction of result set on frontend, not through the API)
	- Timeline
	- Category
- Style rules for markers
- Style rules for lines and polygons 
- Map center point and start zoom level

### F8. Embed

The Data Atlas provides the means to upload the configuration of a map into a static HTML file, hosted on Amazon S3 for easy embedding. It is a separate website, hosted by OneWorld, built on top of the open source content management system Drupal. 

The following features will be added to the Data Atlas to support and extend the ErfGeoviewer:

- Currently the Data Atlas only supports a world map. A new option will be added to use the ErfGeoviewer.
- A configuration form (defined in Feature 7) will be created to support the viewer.
- Maps can be configured inside of the Atlas. A live preview is shown to show the effect of various configuration options.
- Maps are associated with a user account. This provides easy access to edit an existing map.
- Maps are published to the Amazon Cloud in the form of a static HTML file containing references to the ErfGeoviewer.

A separate section about the Data Atlas can be found later in this document.

### F9. Cloning

- Users can clone an existing map into their own Data Atlas account
- This creates an exact copy, which can be modified to fit the user's requirements
- This feature is handled by the Data Atlas, not by the ErfGeoviewer itself.
- All maps contain a "clone" action which links to the Data Atlas.

### F10. Timeline Filter

A slider will allow users to filter a layer based on a date range. 

- The timeline filter can target one or more layers.
- The filter requires a start and end date. 
- Filtering reduces what is shown on the frontend. It does not alter the query used to fetch the data from the API.

### F11. Category Filter

Functionality not yet determined.

### F12. Search field

This feature cannot be specified until the Zoek en Vind API has been designed. I am imaging a simple text field (similar to Google) that searches a predefined collection by keyword.

### F13. Search result list

The results of the search are listed in a pane overlay before being plotted on a map. 

### F14. Map layers

A single map may require multiple result sets separated into different layers. For example, a map of windmills would be one query placed in a single layer, likely given its own icon and styling; additionally, a layer showing recreational areas would require a second query to the Zoek en Vind API, 

### F15. Layouts

The application will be designed using Marionette regions and underscore templates. This provides a flexible way to adjust the presentation of a map instance.

Components of the map
-------------------------------

This section describes the map components of the ErfGeoviewer. Most of these can be toggled on or off through configuration settings. This is useful, for example, when a map instance is only intended to show a single result, where panning, zooming or further searching is not desired.

### Zoom controls

Users can zoom using a mouse wheel, by pinching-in and pinching-out on a touch device, or by clicking on a plus and minus icon overlaid on the map. The zoom control also contains a toggle for entering fullscreen mode.

### Legend

Different datasets will have different needs for describing the objects on a map. An array of icons with labels can be configured based on the fields. This is configured at either an application or instance level (see section "Configuration").

### Base tile layer

A default tile set will be served to show country borders, city and town names, streets and rivers. Our preference is to work with tile services provided by [Mapbox](https://www.mapbox.com/), which offers a lot of flexibility for styling and is free for low traffic sites. However, this can easily be swapped with public tile servers such as OpenStreetMap. To be clear, use of a commercial tile service such as Mapbox would **not** be a dependency of the application.

### Additional tile layers

Some maps may need to show more than one tile layer. For example, a historical map may need to be overlaid on a present day transportation map to provide spatial context. 

### GeoJSON features

Polylines and polygons can be drawn using leaflet. The API should return GeoJSON for rendering these shapes on a map.

### Search

The ErfGeoviewer will have search capabilities within the application itself. This component manages the search field.

### Results

Results can be seen in two ways: either in a paginated list overlaid on the map, or within the map in the form of markers or polygons. When multiple markers are placed in the same location, they will be clustered together. The zoom level and center point will be adjusted to fit all search results shown on the map.

### Filter

Search results can be filtered. This component will add additional parameters to the search in order to reduce its results. For data with time series, the [Crossfilter](http://square.github.io/crossfilter/) library for JavaScript is an ideal option.

### Popups

An object on the map can be inspected for more information. This could take the form of a sidebar display, or as a popup. The design will be determined at a later stage, and be based on the expected amount of content to be displayed. A popup is suitable for small amounts of information such as a title, while sidebars and overlays are easier for digesting larger amounts of information.

### Embed tool

Users can copy the link of the current page, or embed the map instance as an iframe.

ErfGeoviewer configuration
-----------------------------------

The ErfGeoviewer will have two layers of configuration. The first, the application configuration, is intended to provide basic settings to fit a particular website. This is where the connection to a dataset or multiple datasets is defined, and the way in which we can customize the viewer to fit the look and feel of the site that hosts it.

### Application configuration

The base configuration of the application is maintained in JSON format. Its parameters will govern both the behavior and the look of the application. Sample parameters include:

- Link to logo
- Primary color
- URL to tile server
- Center point of the map
- Zoom min, max and start level
- E&L database(s)

In addition, map components can be enabled or disabled:

- Legend
- Search box
- Filtering
- Popups
- Zoom controls

### Instance configuration

An ErfGeoviewer "instance" is a single occurrence of a map within a website. It uses the application configuration as a default, then overrides some or all of its parameters to display a particular collection or item. In general, a website would have a single application configuration object, and each map shown in the website would be an instance.

Any setting from the application configuration can be overridden by an instance. 

- *All application settings*, plus:
- Title
- Description
- Parameters to retrieve a set of documents: fields, collections, filters
- Fields to show when clicking on a map feature such as a marker
- Labels for fields

### Creating a map

The Data Atlas was designed to create these configuration files without knowledge of JavaScript. In fact, a user needs to know nothing about "configuration files": they use a simple web form to adjust the way a map looks or behaves, then publish it to receive an embed code. The embed code contains everything it needs to show a map instance. 

Data Atlas
---------------

### Case study

Since the initial launch of the Data Atlas, OneWorld has published over 200 public datasets and 100 visualizations. That's quite an accomplishment. International data on pollution, land grabbing, poverty, employment and income has been assembled, mapped and charted on the OneWorld site. 

The tool proved very useful for OneWorld, and they wanted to expand the tool and offer it to journalists from other organizations and newspapers. The tool, however, was a module written for a specific content management system (CMS), Drupal, and had too many dependencies on the OneWorld site. Its usage was too narrow for a general public.

We chose instead to separate the Atlas into a separate, much smaller site that journalists could use to log in, upload their data, and create maps. Once the map is created, it is completely detached from the tool created it, the way a text file is detached from the text editor that creates it. That ensures that the map is highly embeddable with only frontend components: an HTML document, a stylesheet, JavaScript to render the map, and a CSV file as the source of the data. We combined these elements together and host them in the cloud, and make them available for in an iFrame.

### Components relevant to ErfGeoviewer

- Configuring a map
- Publishing to the cloud for easy embedding 
- The project architecture

Layouts
-----------

Not all datasets are the same, so layouts are an important aspect of fitting different use cases of the ErfGeoviewer.

The following map components can be configured to appear in a certain corner of the map (top left, top right, bottom left, bottom right):

- Legend
- Search 
- Zoom
- Layer toggles
- Embed button

The list of results and the popups are configured using stylesheets. In make the layout easier to adjust, we separate into SASS "partials". This is a structural way of separating styling for layout from the styling of buttons, typography, color, markers, polygons and other features. 


----

# Section 2: External components

This section lists major open source projects that will be used to build the ErfGeoviewer, and the API's that will be used to access data presented by the ErfGeoviewer.

JavaScript libraries
---------------------

The ErfGeoviewer will be built using open source tools, primarily written in JavaScript. The end product will be a front-end JavaScript application. Backend components are entirely separated. A separated API will be used for querying and retrieving documents, and a Node.js server can be optionally used by contributors to facilitate faster development and manage dependencies. 

- Leaflet.js - a versatile, widely adopted mapping library. (Github stars: 10,492)
- D3.js - visualization library with substantial support for calculating geographic projections and rendering them. It can be used in combination with Leaflet to create highly customized maps. (Github stars: 37,424)
- Backbone.js - light MVP framework with few dependencies, used to structure code. (Github stars: 21,673)
- Marionette.js - extension to Backbone to facilitate common design patterns in the application structure. (Github stars: 6,139)
- Bower - package manager for managing application dependencies. Used only for development. (Github stars: 12,191)
- Grunt - task manager to streamline common tasks such as bundling software (Github stars: 9,453)

Other tools and libraries are likely to be used, but the aforementioned six are the most important. A Github star indicates the popularity of a project, and is something between a bookmark in a web browser and a "like" on Facebook. A project with more than one hundred stars can be seen as popular or maturing, and above one thousand stars often signals an industry standard. The projects listed above are massively popular, with the smallest being Marionette at over six thousand.

### Differences from ErfGeoviewer prototype

The ErfGeoviewer prototype developed by de Waag was built using some of the same tools. Two differences are worth noting. First, we propose using Leaflet instead of OpenLayers as the primary mapping tool. Leaflet is significantly smaller and encompasses all the expected functionality needed by the ErfGeoviewer. Secondly, we propose Backbone.js instead of Angular.js. Both have major support in the open source community, but our developers have more experience with Backbone.js and can reuse some of their existing code.

### How these components work together

Leaflet.js is the primary mapping tool of the ErfGeoviewer. It is used to render tiles, markers, geographic shapes (markers, polygons, lines), handling user interactions such as swiping the map, zooming, setting the center location, and displaying pop-ups. It supports mobile and touch devices, has a pluggable architecture and a lively open source community.

D3.js is a "lower level" tool for manipulating data, and includes libraries useful in drawing maps based on GeoJSON and provides functions for calculating spatial areas and distances. It is also a powerful tool for data visualization, and excels in making charts, animations and manipulating datasets. It is a natural partner to Leaflet when specialized behaviors or visuals are necessary. It can draw shapes on top of a map rendered by Leaflet, or render charts inside a popup that's triggered by a Leaflet event.

Backbone.js and Marionette.js assist in creating separation of concern within the application. Presentation logic is structurally isolated from code that draws data from the API, and also from the "business logic" that orchestrates the steps of rendering the map: initializing libraries, requesting data, processing a user search, and passing search results to the map for display. 

Bower and Grunt are used by developers only, and will not be bundled into the ErfGeoviewer that is embedded on websites. However, they are important to mention because they influence both the workflow of the project and its architecture.

Bower is a package manager. The ErfGeoviewer describes all of its dependencies such as Leaflet and D3 in a single file. This file is placed under version control, in git. When contributors of a project begin setting up their local environment, they often begin by looking at this file. It shows very clearly which tools are in use, which versions, and how they depend on one another. The file also provides the means to download all of these dependencies, and relieves 

Grunt is a task manager. It helps running common or frequent tasks such as starting a server, or compiling SASS files into CSS when a file change is detected. It is also used for compiling the many JavaScript libraries used by the application into a single, minimized file with a reduced size. This improves performance and makes it easier to embed into a website.



E&L API requirements
-----------------------

Definition of the API should be a collaborative project between frontend and backend developers. This document is not an exhaustive list of requirements and does not consider issues such as authentication, pagination or ordering. The exact data model is not known; this is a best guess of the structure. 

Results returned by the API endpoint should be in JSON format (JavaScript Object Notation), and are assumed to be provided via a REST (Representational state transfer) interface. 

Ideally, relationships between objects can also be shown. For example, when viewing a Story, it would be interesting to see links to related Events or Videos. To describe linked data, JSON-LD (Linked Data) is recommended. 

### Querying

We propose that the API be based upon [Search/Retrieval via URL (SRU)](http://www.loc.gov/standards/sru/sru-1-2.html) version 1.2. This is used by the Digitale Collectie and the Library of Congress. 

### Adaptors

Various collections are accessed using different API's, which have different structures for querying and return differently formatted results. Adapters will be required to create a single entry point into various collections.

### Results

The format of the results are not yet known.

### Record types

- Story: an article with internal references to images and marked up text.
- Event: a historical event (such as a battle) or upcoming activity (museum showing or a tour)
- Image: a single image in JPG or PNG format.
- Video: a reference to YouTube or Vimeo.
- Collection: a list containing elements listed above.

### Child elements

- Location: the longitude and latitude center point of the object.
- Area: if the object includes a geographical area, this should be expressed in GeoJSON.
- Tags: used for categorizing an object.

### Sample filters

- search: free text search on any of the object's child elements (title, description, etc).
- keyword: Filters on taxonomy term (tags).
- type: Filters on object type (person, city, story).
- nearby: Filters on proximity to a geographical point.
- locality: Filters on location name, such as a town or city.
- bounding box: Filters results to only those that fit within the given bounding area.
- before, after: Filters on date.

Aansluiting: Delving
----------------------------

Total Active Media will create a small PHP service used to access Delving and present results. This will be done while the Zoek en Vind API is still under development.

It has not yet been determined who will create an adaptor so that Delving can be queried through the Zoek en Vind API, and the XML results of Delving are converted to JSON-LD.

Aansluiting: Digitale Collectie
-----------------------------------------

De Waag will create an adaptor for accessing the Digitale Collectie via the Zoek en Vind API.


Aansluiting: kadastrale kaarten (watwaswaar.nl)
-----------------------------------------------------------------------

Because this service has not yet been created, it is not possible to specify how the ErfGeoviewer will access it.

The ErfGeoviewer will support this collection by adding additional add-on modules to the ErfGeoviewer. The additional modules will be maintained in a separate repository, since they are not required by the core package and are not relevant to all developers who wish to work with or install the ErfGeoviewer.

- A specialized backend connector to fetch results from a kadastrale kaarten web service
- A specialized layout for showing ErfGeoviewer results

In the "Nota van Inlichtingen", this was stated:

> Het is de bedoeling dat de verschillende databronnen op zoveel mogelijk dezelfde wijze bevraagd kunnen worden via één centrale API, zodat de response uniform is. Mogelijk zijn onder de centrale API wel API’s per collectiesoort/ aggregator nodig; dat wordt nog onderzocht buiten de huidige opdracht E&L Webvisualisatie.

Having a single API to retrieve all results is highly desirable, but we shouldn't be too optimistic that all the necessary datasets can be aggregated into a single central API by the time this launches. This collection in particular may require certain exceptions until it can be brought into the central system. 

----

# Section 3: Development Practices

This section describes the process that will be followed during development, and the tools used for collaboration.

Open Source
----------------

Open source software allows for the free movement of knowledge, tools and software to build flexible and sustainable solutions. It has a number of advantages over closed source solutions:

### Audibility

Because the source is open, it can be audited by anyone. When dealing with proprietary systems, this is something one has to take the vendor's word for it.

### Quality

Open source software projects tend to get better when the community of both users and developers grow. Also as a result of the Audibility open source software generally is of higher quality than proprietary alternatives.

### Open standards

Open source software is developed based on open standards. This means interfacing with it from other systems is generally easier than with proprietary systems.

### Security

The open nature of the source code and its audibility also makes it possible to be audited by security professionals. Also open source projects patch software more quickly than proprietary ones when a vulnerability is discovered.

### Cost

Because of the size of open source projects and their communities it is generally cheaper to develop software extending existing open source software. Otherwise building software in an open source way in order to spread cost.

Development Process
------------------------------

### Project Management System

We make extensive use of project management systems, which suits agile processes. This helps us keep track of features as well as bug reports. For internal and private projects we make use of Redmine, which makes use of some of the same ticketing and tracking conventions as Github. For a public project, bug reporting and tracking of milestones would happen on Github.

### Version Control

We keep our software under version control using Git and use the Git flow branching model to facilitate different modes of operation (i.e. maintenance or new features). For this project, all code would be publicaly hosted on Github.

### DTAP

Our environments are split up in order to facilitate the different modes of operation. A development environment is available per developer for developing the software. A test environment is available for internal and automated testing purposes. And an acceptance environment is available for user acceptance testing (UAT). A production environment is used for actually running the software for its end-users.

###  Best practices

We make use of known design patterns to avoid reinventing the wheel. We use existing, well supported architectures, frameworks and components when they exist.

### Quality Assurance

Our code is reviewed by an automated system before being deployed to a test/acceptance or production environment. This review makes sure no syntax and code style errors will be deployed.


Webrichtlijnen 2.0
----------------------

### What is it?

We aim for full compliance with [Webrichtlijnen versie 2](http://versie2.webrichtlijnen.nl/). 

### Approach

In the same way that developing for older versions of Internet Explorer requires certain certain work-arounds, making highly accessible applications also requires tricks of the trade. The developers at the Total Identity group keep close contact to share tips and 

When possible, the application will take advantage of existing accessibility features of the web browser. For example, by using standard form elements instead of highly customized input boxes or drop downs, the browser can render these known elements based on the needs of the viewer.

