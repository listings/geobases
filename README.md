GeoBases
========

Introduction
------------

This module is a general class *GeoBase* to manipulate geographical
data. It loads static csv files containing data about airports or train
stations, and then provides tools to play with it. This is entirely
written in Python.

It relies on three other modules:

-   *GeoUtils*: to compute haversine distances between points
-   *LevenshteinUtils*: to calculate distances between strings. Indeed,
    we need a good tool to do it, in order to recognize things like
    station names in schedule files where we do not have the station id
-   *GeoGridModule*: a class implementing a geographical index based on
    geohashing

Project
-------

### Prerequisites

These prerequisites are very standard packages which are often installed
by default on Linux distributions. But make sure you have them anyway.

First you need to install *setuptools*:

    # apt-get install python-setuptools    # for debian
    # yum install python-setuptools.noarch # for fedora

Then you need some basics compilation stuff to compile dependencies:

    # apt-get install python-dev g++    # for debian
    # yum install python-devel gcc-c++  # for fedora

### Installation

To clone the project from
[github](https://github.com/opentraveldata/geobases.git)\_:

    % git clone https://github.com/opentraveldata/geobases.git

Then install the package:

    % cd geobases
    % python setup.py install --user

This should install some dependencies.

A standalone script is put in `~/.local/bin`, to benefit from it, put
that in your `~/.bashrc` or `~/.zshrc`:

    export PATH=$PATH:$HOME/.local/bin
    export BACKGROUND_COLOR=black # or 'white', depending on your terminal configuration

If you use zsh and want to benefit from the *autocomplete*, add this to
your `~/.zshrc`:

    # Add custom completion scripts
    fpath=(~/.zsh/completion $fpath)
    autoload -U compinit
    compinit

Tests
-----

You may try to run the tests with:

    % find ./ -name '*.pyc' -exec rm {} \;
    % python test/test_GeoBases.py -v

Quickstart
----------

To load the class, just import the main class with:

    % python
    >>> from GeoBases import GeoBase
    >>> geo_o = GeoBase(data='ori_por', verbose=False)
    >>> geo_a = GeoBase(data='airports', verbose=False)
    >>> geo_t = GeoBase(data='stations', verbose=False)

You may provide other values than *data="ori\_por"*,
*data="airports\_csv"* or *data="stations"*. Here is an overview:

-   *data="ori\_por"* will load a local version of this
    [file](https://github.com/opentraveldata/optd/raw/trunk/refdata/ORI/ori_por_public.csv)\_
-   *data="ori\_por\_multi"* is the same as previous, but the key for a
    line is not the iata\_code, but the concatenation of iata\_code and
    location\_type. This feature makes every line unique, whereas
    *ori\_por* may have several lines for one iata\_code, and duplicates
    are dropped. \_\_id\_\_ is the special field containing the key.
-   *data="airports"* will use geonames as data source for airports
-   *data="airports\_csv"* will use an airports data source
-   *data="stations"* will use RFF data, from [the open data
    website](http://www.data.gouv.fr)\_, as data source for french train
    stations
-   *data="stations\_nls"* will use NLS nomenclature as data source for
    french train stations
-   *data="stations\_uic"* will use UIC nomenclature as data source for
    french train stations
-   *data="countries"* will load data on countries
-   *data="capitals"* will load data on countries capitals
-   *data="continents"* will load data on continents
-   *data="timezones"* will load data on timezones
-   *data="languages"* will load data on languages
-   *data="cities"* will load data on cities, extracted from geonames
-   *data="currencies"* will load data on currencies, extracted from
    wikipedia
-   *data="airlines"* will load data on airlines, extracted from
    CRB\_Airline
-   *data="cabins"* will load data on cabins
-   *data="regions"* will load data on regions
-   *data="locales"* will load data on locales
-   *data="location\_types"* will load data on location types
-   *data="feature\_classes"* will load data on feature classes
-   *data="feature\_codes"* will load data on feature codes
-   *data="ori\_por\_non\_iata"* will load some non-iata data excluded
    from *ori\_por*
-   *data="geonames\_MC"* will load MC data of geonames
-   *data="geonames\_FR"* will load FR data of geonames
-   *data="postal\_codes\_MC"* will load MC postal codes data
-   *data="postal\_codes\_FR"* will load FR postal codes data
-   *data="feed"* will create an empty instance

All features are then data independent, and are available as long as
geocodes are included in the data sources (which is not the case for
countries or NLS nomenclature).

Features
--------

### Information access

    >>> geo_a.get('CDG', 'city_code')
    'PAR'
    >>> geo_a.get('BRU', 'name')
    'Bruxelles National'
    >>> geo_t.get('frnic', 'name')
    'Nice-Ville'
    >>>
    >>> geo_t.get('frnic', 'not_a_field')
    Traceback (most recent call last):
    KeyError: "Field 'not_a_field' [for key 'frnic'] not in ['info', 'code', 'name', 'lines@raw', 'lines', '__gar__', '__par__', '__dup__', '__key__', 'lat', 'lng', '__lno__']"
    >>> geo_t.get('fr_not_exist', 'name', default='NAME')
    'NAME'

### Find airports with properties

    >>> conditions = [('city_code', 'PAR'), ('location_type', 'H')]
    >>> list(geo_o.getKeysWhere(conditions, mode='and'))
    ['JDP', 'JPU']
    >>> conditions = [('city_code', 'PAR'), ('city_code', 'LON')]
    >>> len(list(geo_o.getKeysWhere(conditions, mode='or')))
    36

### Distance calculation

    >>> geo_a.distance('CDG', 'NCE')
    694.5162...

### Find airports near a point

    >>> # Paris, airports <= 50km
    >>> [geo_a.get(k, 'name') for d, k in sorted(geo_a.findNearPoint((48.84, 2.367), 50))]
    ['Paris-Orly', 'Paris-Le Bourget', 'Toussus-le-Noble', 'Paris - Charles-de-Gaulle']
    >>>
    >>> # Nice, stations <= 5km
    >>> [geo_t.get(k, 'name') for d, k in sorted(geo_t.findNearPoint((43.70, 7.26), 5))]
    ['Nice-Ville', 'Nice-Riquier', 'Nice-St-Roch', 'Villefranche-sur-Mer', 'Nice-St-Augustin']

### Find airports near a key

    >>> sorted(geo_a.findNearKey('ORY', 50)) # Orly, airports <= 50km
    [(0.0, 'ORY'), (18.8..., 'TNF'), (27.8..., 'LBG'), (34.8..., 'CDG')]
    >>> sorted(geo_t.findNearKey('frnic', 5)) # Nice station, stations <= 5km
    [(0.0, 'frnic'), (2.2..., 'fr4342'), (2.3..., 'fr5737'), (4.1..., 'fr4708'), (4.5..., 'fr6017')]

### Find closest airports from a point

    >>> list(geo_a.findClosestFromPoint((43.70, 7.26))) # Nice
    [(5.82..., 'NCE')]
    >>> list(geo_a.findClosestFromPoint((43.70, 7.26), N=3)) # Nice
    [(5.82..., 'NCE'), (30.28..., 'CEQ'), (79.71..., 'ALL')]
    >>> list(geo_t.findClosestFromPoint((43.70, 7.26), N=1)) # Nice
    [(0.56..., 'frnic')]
    >>> list(geo_t.findClosestFromPoint((43.70, 7.26), N=2, from_keys=('frpaz', 'frply', 'frbve'))) # Nice
    [(482.84..., 'frbve'), (683.89..., 'frpaz')]

### Approximate name matching

    >>> geo_t.fuzzyGet('Marseille Charles', 'name')[0]
    (0.8..., 'frmsc')
    >>> geo_a.fuzzyGet('paris de gaulle', 'name')[0]
    (0.78..., 'CDG')
    >>> geo_a.fuzzyGet('paris de gaulle', 'name', approximate=3)
    [(0.78..., 'CDG')]
    >>> geo_t.fuzzyGetCached('Marseille Saint Ch.', 'name')[0] # Cached for further calls
    (0.8..., 'frmsc')

### Map display

    >>> geo_t.visualize()
    * Added lines for duplicates linking, total 0
    > Affecting category None     to color blue    | volume 3190
    <BLANKLINE>
    * Now you may use your browser to visualize:
    example_map.html example_table.html
    <BLANKLINE>
    * If you want to clean the temporary files:
    rm example.json ...
    <BLANKLINE>
    (['example_map.html', 'example_table.html'], 2)

![image](https://raw.github.com/opentraveldata/geobases/public/examples/GeoBases-map.png)

Standalone script
-----------------

Installation of the package will also deploy a standalone script under
the name GeoBase.

If you use zsh and want to benefit from the *autocomplete*, add this to
your `~/.zshrc`:

    # Add custom completion scripts
    fpath=(~/.zsh/completion $fpath)
    autoload -U compinit
    compinit

Then you may use:

    % GeoBase ORY CDG
    % GeoBase --closest CDG
    % GeoBase --near LIG
    % GeoBase --fuzzy marseille
    % GeoBase --help

![image](https://raw.github.com/opentraveldata/geobases/public/examples/GeoBases-CLI.png)

French riviera entries:

    % GeoBase -E adm1_code -e B8

Same with programmer-friendly output (csv-like):

    % GeoBase -E adm1_code -e B8 --quiet --show __ref__ iata_code  name

Add a fuzzy name search:

    % GeoBase -E adm1_code -e B8 --fuzzy sur mer

All heliports under 200 km from Paris:

    % GeoBase --near PAR -N 200 -E location_type -e 'H'

50 train stations closest to Paris:

    % GeoBase -E location_type -e R --closest PAR -C 50  --quiet --show iata_code name

Countries with non-empty postal code regex:

    % GeoBase -b countries -E postal_code_regex -e "" --reverse --quiet

Reading data input on stdin:

    % echo -e 'ORY^Orly\nCDG^Charles' |GeoBase

Display on map:

    % GeoBase -b stations --map

Europe marker-less map:

    % GeoBase -E region_code -e EUROP -m -M _ _ country_code  __none__

If the previous commands fail, it might be because you PATH does not
include the local bin directory, and you installed the package in user
space:

    % export PATH=$PATH:$HOME/.local/bin
    % export BACKGROUND_COLOR=black # or 'white', depending on your terminal configuration

Packaging
---------

To create source distribution (pip-installable):

    % python setup.py sdist --format=zip

To create rpm packages:

    % rm -rf build dist *.egg-info
    % python setup.py bdist_rpm

The `MANIFEST.in` file is used to determine which files will be included
in a source distribution.

`package_data` directive in `setup.py` file is about which file will be
exported in site-package after installation.

So you really need both if you want to produce installable packages like
rpms or zip which can be installed afterwards.

You will also find a [Rakefile](http://rake.rubyforge.org/)\_ at the
root of the project. This may be used to build and deploy the packages.
The Rakefile

Deployment is done using webdav, and the Rakefile expects `nd` to be
installed (this is a webdav client).

To install nd, fetch the sources from
[<http://www.gohome.org/nd/>](http://www.gohome.org/nd/)\_. Then compile
and install them. On 64 bits Fedora, you need to install libxml2 before:

    # yum install libxml2.x86_64 libxml2-devel.x86_64

After nd and rake installation, you may try:

    % rake

Virtualenv has bugs on 64 bits systems, if you are using such a system,
you absolutely need to upgrade to the very last unreleased version of
virtualenv, before executing rake:

    % pip uninstall virtualenv
    % pip install --user https://github.com/pypa/virtualenv/tarball/develop
