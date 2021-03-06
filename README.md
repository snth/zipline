Zipline
=======

Zipline is a financial backtester for trading algorithms written in
Python. The system is fundamentally event-driven and a close
approximation of how live-trading systems operate.

Zipline is currently used in production as the backtesting engine
powering <https://app.quantopian.com> -- a free, community-centered
platform that allows development and real-time backtesting of trading
algorithms in the web browser.

Features
========

* Ease of use: Zipline tries to get out of your way so that you can
focus on algorithm development. See below for a code example.

* Zipline comes "batteries included" as many common statistics like
moving average and linear regression can be readily accessed from
within a user-written algorithm.

* Input of historical data and output of performance statistics is
based on Pandas DataFrames to integrate nicely into the existing
Python eco-system.

* Statistic and machine learning libraries like matplotlib, scipy,
statsmodels, and sklearn support development, analysis and
visualization of state-of-the-art trading systems.

Installation
============

Since zipline is pure-python code it should be very easy to install
and set up with pip:

```pip install zipline```

If there are problems installing the dependencies or zipline we
recommend installing these packages via some other means. For Windows,
the [Enthought Python Distribution](http://www.enthought.com/products/epd.php)
includes most of the necessary dependencies. On OSX, the [Scipy Superpack]
(http://fonnesbeck.github.com/ScipySuperpack/) works very well.

Dependencies
------------

* Python (>= 2.7.2)
* numpy (>= 1.6.0)
* pandas (>= 0.9.0)
* pytz
* msgpack-python
* iso8601
* Logbook
* blist

Quickstart
==========

The following code implements a simple dual moving average algorithm
and tests it on data extracted from yahoo finance.

```python
from zipline.algorithm import TradingAlgorithm
from zipline.transforms import MovingAverage
from zipline.utils.factory import load_from_yahoo

class DualMovingAverage(TradingAlgorithm):
    """Dual Moving Average algorithm.
    """
    def initialize(self, short_window=200, long_window=400):
        # Add 2 mavg transforms, one with a long window, one
        # with a short window.
        self.add_transform(MovingAverage, 'short_mavg', ['price'],
                           market_aware=True,
                           days=short_window)

        self.add_transform(MovingAverage, 'long_mavg', ['price'],
                           market_aware=True,
                           days=long_window)

        # To keep track of whether we invested in the stock or not
        self.invested = False

        self.short_mavg = []
        self.long_mavg = []


    def handle_data(self, data):
        if (data['AAPL'].short_mavg['price'] > data['AAPL'].long_mavg['price']) and not self.invested:
            self.order('AAPL', 100)
            self.invested = True
        elif (data['AAPL'].short_mavg['price'] < data['AAPL'].long_mavg['price']) and self.invested:
            self.order('AAPL', -100)
            self.invested = False

        # Save mavgs for later analysis.
        self.short_mavg.append(data['AAPL'].short_mavg['price'])
        self.long_mavg.append(data['AAPL'].long_mavg['price'])

data = load_from_yahoo()
dma = DualMovingAverage()
results = dma.run(data)
```

You can find other examples in the zipline/examples directory.

Style Guide
===========

To ensure that changes and patches are focused on behavior changes,
the zipline codebase adheres to PEP-8,
<http://www.python.org/dev/peps/pep-0008/>.

The maintainers check the code using the flake8 script,
<https://github.com/jcrocholl/pep8/>, which is included in the
requirements_dev.txt.

Before submitting patches or pull requests, please ensure that your
changes pass ```flake8 --ignore=E124,E125,E126 zipline tests```

Discussion and Help
===================

Discussion of the project is held at the Google Group,
<zipline@googlegroups.com>,
<https://groups.google.com/forum/#!forum/zipline>.

Source
======

The source for Zipline is hosted at
<https://github.com/quantopian/zipline>.

Build Status
============

[![Build Status](https://travis-ci.org/quantopian/zipline.png)](https://travis-ci.org/quantopian/zipline)

Contact
=======

For other questions, please contact <opensource@quantopian.com>.
