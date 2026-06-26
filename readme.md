# HAMON

HAMON is a passive optical sequence-mixing model for long-horizon time-series forecasting.

The model encodes a time-series history onto an optical aperture, leaves future positions dark, and uses trainable phase masks with free-space propagation to produce the forecast field. The forecasting core is the optical propagation operator. Digital computation is used only at the boundary for normalization, input encoding, readout calibration, denormalization, and training.

## Main idea

HAMON treats forecasting as learned diffraction.

Observed history is written into an input field. The future region starts dark. A stack of trainable phase masks shapes the propagated optical field so that the forecast appears in the future region.

The model does not use a digital temporal forecasting head after the optical core.

## Model options

HAMON has two input encodings.

Amplitude encoding:

    ENCODING_MODE = "amplitude"

The normalized history directly sets the input field amplitude.

Phase encoding:

    ENCODING_MODE = "phase"
    PHASE_ALPHA = 0.3

The normalized history sets the input optical phase while the input magnitude remains approximately fixed.

HAMON has three readout modes.

Coherent readout:

    READOUT_MODE = "coherent"

Reads the real field quadrature in the forecast region.

Intensity readout:

    READOUT_MODE = "intensity"

Reads detector intensity, |U|^2.

Differential intensity readout:

    READOUT_MODE = "differential_intensity"

Uses two forecast regions and subtracts their detected intensities. This gives a signed output using intensity-compatible detection.

## Common configurations

Amplitude/coherent HAMON:

    ENCODING_MODE = "amplitude"
    READOUT_MODE = "coherent"
    USE_BACKCAST_LOSS = True
    USE_INTENSITY_AFFINE = False

Phase/coherent HAMON:

    ENCODING_MODE = "phase"
    READOUT_MODE = "coherent"
    PHASE_ALPHA = 0.3
    USE_BACKCAST_LOSS = True
    USE_INTENSITY_AFFINE = False

Phase/intensity HAMON:

    ENCODING_MODE = "phase"
    READOUT_MODE = "intensity"
    PHASE_ALPHA = 0.3
    USE_BACKCAST_LOSS = False
    USE_INTENSITY_AFFINE = True

Phase/differential-intensity HAMON:

    ENCODING_MODE = "phase"
    READOUT_MODE = "differential_intensity"
    PHASE_ALPHA = 0.3
    USE_BACKCAST_LOSS = False
    USE_INTENSITY_AFFINE = True

## Optical depth

The main experiments use 16 trainable phase-mask layers.

    LAYER_CANDIDATES = [16]

Some interface ablations use 4 layers.

    LAYER_CANDIDATES = [4]

## Errata and reproduction status

After the first arXiv release, we found a validation/test windowing mismatch relative to Time-Series-Library.

The original code used the correct chronological train/validation/test split lengths and fit the scaler only on the training split. However, validation and test arrays were sliced as isolated segments. This omitted the first SEQ_LEN forecast origins from validation and test evaluation.

Original slicing:

    train = data[:train_len]
    val = data[train_len : train_len + val_len]
    test = data[train_len + val_len :]

Corrected Time-Series-Library-compatible slicing:

    train = data[:train_len]
    val = data[train_len - SEQ_LEN : train_len + val_len]
    test = data[train_len + val_len - SEQ_LEN : train_len + val_len + test_len]

This is not train/test leakage. The added context consists only of past observations used as input history. Prediction targets remain inside the validation/test regions, and the scaler is still fit only on the training split.

The HAMON architecture and scientific mechanism are unchanged. The correction affects benchmark windowing and exact reported numbers, not the optical sequence-mixing core.

Corrected HAMON runs are in progress. Early corrected results are broadly consistent with the original conclusions, and some horizons improve, but final corrected tables will be released with arXiv v2.

We are also reproducing FITS inside the same pipeline as an additional sanity check.

Until arXiv v2 is posted, please treat the first-version tables as preliminary and use the corrected repository results when available.
