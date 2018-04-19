Tracking machine learning challenge (TrackML) utility library
=============================================================

A small library to simplify working with the dataset of the tracking machine
learning challenge.

Installation
------------

The package can be installed as a user package via

    pip install --user <path/to/repository>

To make a local checkout of the repository available directly it can also be
installed in development mode

    pip install --user --editable .

In both cases, the package can be imported via `import trackml` without
additional configuration. In the later case, changes made to the code are
immediately visible without having to reinstall the package.

Usage
-----

To read the data for one event from the training dataset including the ground
truth information:

```python
from trackml.dataset import load_event

hits, cells, particles, truth = load_event('path/to/event000000123')
```

For the test dataset only the hit information is available. To read only this
data:

```python
from trackml.dataset import load_event

hits, cells = load_event('path/to/event000000456', parts=['hits', 'cells'])
```

To iterate over events in a dataset:

```python
from trackml.dataset import load_dataset

for event_id, hits, cells, particles, truth in load_dataset('path/to/dataset'):
    ...
```

Each event is lazily loaded during the iteration. Options are available to
read only a subset of available events or only read selected parts, e.g. only
hits or only particles.

To generate a random test submission from truth information and compute the
expected score:

```python
from trackml.randomize import shuffle_hits
from trackml.score import score_event

shuffled = shuffle_hits(truth, 0.05) # 5% probability to reassign a hit
score = score_event(truth, shuffled)
```

All methods either take or return `pandas.DataFrame` objects. Please have a look
at the function docstrings for detailed documentation.

Authors
-------

*   David Rousseau
*   Ilija Vukotic
*   Moritz Kiehn
*   Sabrina Amrouche

License
-------

All code is licensed under the [MIT license][mit_license].

Dataset
-------

A dataset comprises multiple independent events, where each event contains
simulated measurements of particles generated in a collision between proton
bunches at the [Large Hadron Collider][lhc] at [CERN][cern]. The goal of the
tracking machine learning challenge is to group the recorded measurements or
hits for each event into tracks, sets of hits that belong to the same initial
particle. A solution must uniquely associate each hit to one track (although
some hits can be left unassigned). The training dataset contains the recorded
hits, their truth association to particles, and the initial parameters of those
particles. The test dataset contains only the recorded hits.

The dataset is provided as a set of plain `.csv` files ('.csv.gz' or '.csv.bz2'
are also allowed)'. Each event has four associated files that contain hits,
hit cells, particles, and the ground truth association between them.

    event000000000-hits.csv
    event000000000-details.csv
    event000000000-particles.csv
    event000000000-truth.csv
    event000000001-hits.csv
    event000000001-details.csv
    event000000001-particles.csv
    event000000001-truth.csv

Submissions must be provided as a single `.csv` file for the whole dataset with
a name starting with `submission`, e.g.

    submission-test.csv
    submission-final.csv

### Event hits

The hits file contains the following values for each hit/entry:

*   **hit_id**: numerical identifier of the hit inside the event.
*   **x, y, z**: measured x, y, z position (in millimeters) of the hit in
    global coordinates.
*   **volume_id**: numerical identifier of the detector group.
*   **layer_id**: numerical identifier of the detector layer inside the
    group.
*   **module_id**: numerical identifier of the detector module inside
    the layer.

The volume/layer/module id could be in principle be deduced from x, y, z. They
are given here to simplify detector-specific data handling.

### Event hit cells

The cells file contains the constituent active detector cells that comprise each
hit. A cell is the smallest granularity inside each detector module. It is
identified by two channel identifiers that are unique within each detector
module and encode the position. A cell can provide signal information that the
detector module has recorded in addition to the position. Depending on the
detector type only one of the channel identifiers is valid, e.g. for the strip
detectors, and the value might have different resolution.

*   **hit_id**: numerical identifier of the hit as defined in the hits file.
*   **ch0, ch1**: channel identifier/coordinates unique with one module.
*   **value**: signal value information, e.g. how much charge a particle has
    deposited.

### Event particles

The particles files contains the following values for each particle/entry:

*   **particle_id**: numerical identifier of the particle inside the event.
*   **vx, vy, vz**: initial position (in millimeters) (vertex) in global coordinates.
*   **px, py, pz**: initial momentum (in GeV/c) along each global axis.
*   **q**: particle charge (as multiple of the absolute electron charge).
*   **nhits**: number of hits generated by this particle

All entries contain the generated information or ground truth.

### Event truth

The truth file contains the mapping between hits and generating particles and
the true particle state at each measured hit. Each entry maps one hit to one
particle/track.

*   **hit_id**: numerical identifier of the hit as defined in the hits file.
*   **particle_id**: numerical identifier of the generating particle as defined
    in the particles file.
*   **tx, ty, tz** true intersection point in global coordinates between
    the particle trajectory and the sensitive surface.
*   **tpx, tpy, tpz** true particle momentum in the global coordinate system
    at the intersection point.
*   **weight** per-hit weight used for the scoring metric; total sum of weights
    within one event equals to one.

### Dataset submission information

The submission file must associated each hit in each event to one reconstructed
particle track. The reconstructed tracks must be uniquely identified only within
each event.

*   **event_id**: numerical identifier of the event; corresponds to the number
    found in the per-event file name prefix.
*   **hit_id**: numerical identifier of the hit inside the event as defined
    in the per-event hits file.
*   **track_id**: numerical identifier of the truth particle inside the
    event as defined in the per-event truth file.


[cern]: https://home.cern/
[lhc]: https://home.cern/topics/large-hadron-collider
[mit_license]: http://www.opensource.org/licenses/MIT
