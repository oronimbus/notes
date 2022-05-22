# Using the `logging` module in Jupyter Notebooks

1) In your Python file, say `test_log.py`, file specify the following:
```python
import logging
logger = logging.getLogger(__name__)

def testing_log():
    logger.info("Successful log.")
````

2) Then, open up a notebook and type:
```python
import logging
import sys

from test_log import testing_log

logging.basicConfig(
    format='%(asctime)s | %(levelname)s : %(message)s',
    level=logging.INFO,
    stream=sys.stdout
)

testing_log()
```

3) The standard output in the notebook should now display the time (e.g. 2022-05-22 12:00:00), the level (e.g INFO) and the message (i.e. "Successful log").