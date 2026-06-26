# HAMON

Passive optical sequence mixing for long-horizon time-series forecasting.

HAMON maps a time-series history onto an optical aperture, leaves future positions dark, and uses trainable phase masks plus free-space propagation to produce the forecast field. The optical core is the sequence-mixing operator; the digital parts are only the boundary operations such as normalization, encoding, readout calibration, and training.

## What to change

Most experiments are controlled by a few flags.

### 1. Choose input encoding

Use amplitude encoding:

    ENCODING_MODE = "amplitude"

or phase encoding:

    ENCODING_MODE = "phase"
    PHASE_ALPHA = 0.3

Amplitude encoding writes the normalized history directly into the optical field amplitude.

Phase encoding keeps approximately fixed magnitude and writes the normalized history into optical phase.

### 2. Choose readout

Coherent readout:

    READOUT_MODE = "coherent"

Intensity readout:

    READOUT_MODE = "intensity"

Differential intensity readout:

    READOUT_MODE = "differential_intensity"

Coherent readout reads the real field quadrature.

Intensity readout reads detector intensity, |U|^2.

Differential intensity readout uses two forecast regions and subtracts their intensities, giving a signed output with intensity detectors.

### 3. Choose number of optical layers

For the main runs:

    LAYER_CANDIDATES = [16]

For lightweight interface ablations:

    LAYER_CANDIDATES = [4]

The number of layers is the number of trainable optical phase masks.

### 4. Choose datasets and horizons

Main no-Electricity/Traffic run:

    datasets=["etth1", "etth2", "ettm1", "ettm2", "weather"]
    pred_lens=[96, 192, 336, 720]
    seeds=[1, 2, 3]

Large datasets can be run separately:

    datasets=["electricity"]
    datasets=["traffic"]

## Common configurations

Amplitude/coherent baseline:

    ENCODING_MODE = "amplitude"
    READOUT_MODE = "coherent"
    USE_BACKCAST_LOSS = True
    USE_INTENSITY_AFFINE = False

Phase/coherent:

    ENCODING_MODE = "phase"
    READOUT_MODE = "coherent"
    PHASE_ALPHA = 0.3
    USE_BACKCAST_LOSS = True
    USE_INTENSITY_AFFINE = False

Phase/intensity:

    ENCODING_MODE = "phase"
    READOUT_MODE = "intensity"
    PHASE_ALPHA = 0.3
    USE_BACKCAST_LOSS = False
    USE_INTENSITY_AFFINE = True

Phase/differential-intensity:

    ENCODING_MODE = "phase"
    READOUT_MODE = "differential_intensity"
    PHASE_ALPHA = 0.3
    USE_BACKCAST_LOSS = False
    USE_INTENSITY_AFFINE = True

## Recommended corrected run

Use a fresh output folder:

    BASE_DIR = "/content/drive/MyDrive/HAMON_Ultimate_Fixed"

Use a clear tag:

    EXPERIMENT_TAG = make_experiment_tag() + "__no_elec_traffic_Hsel_L16_s123_80_epochs"

Current corrected protocol:

    SEQ_LEN = 336
    EPOCHS = 80
    PATIENCE = 15
    LAYER_CANDIDATES = [16]
    SEEDS = [1, 2, 3]

Run sanity check first:

    RUN_SANITY_CHECK = True

Then run full benchmark:

    RUN_SANITY_CHECK = False

## Errata / reproduction status

The first arXiv version used a validation/test windowing convention that was slightly different from Time-Series-Library.

The original split used correct chronological train/validation/test lengths and fit the scaler only on training data, but validation and test were sliced as isolated segments. This omitted the first SEQ_LEN forecast origins from validation and test evaluation.

Original slicing:

    train = data[:train_len]
    val = data[train_len : train_len + val_len]
    test = data[train_len + val_len :]

Corrected Time-Series-Library-compatible slicing:

    train = data[:train_len]
    val = data[train_len - SEQ_LEN : train_len + val_len]
    test = data[train_len + val_len - SEQ_LEN : train_len + val_len + test_len]

This is not data leakage. The added context points are only past observations used as input history. Prediction targets remain inside validation/test, and the scaler is still fit only on training data.

The optical model and scientific mechanism are unchanged. The correction affects benchmark windowing/comparability, not the HAMON architecture.

Corrected HAMON reruns are in progress. Early corrected runs are broadly consistent with the original conclusions, and some horizons improve, but final corrected tables will be released in arXiv v2 after the reruns finish.

We are also reproducing FITS inside the same pipeline as an additional sanity check.

Until v2 is posted, please treat the initial arXiv tables as preliminary and use the corrected repository results when available.
