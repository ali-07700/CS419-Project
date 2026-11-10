# CS 419 — Post-earthquake change detection

Group: Ali Yılmaz, Furkan Gönen, Mehmet Ali Bağıbala, Pelin Durukan, Zeynep Okkıran.

Paired satellite images are compared inside known building polygons. Five scores are computed separately, then combined once, with weights chosen on non-earthquake events. Mexico City and Palu are scored once, after those weights are frozen.

The images are the xView2 challenge training set. They are not stored in this repository. Place that download next to this folder, as `../Dataset`, and do not commit it.

On this machine the training release is `../Dataset/train`. Each pair is `images/{id}_pre_disaster.png`, `images/{id}_post_disaster.png`, and `labels/{id}_post_disaster.json`. Pixel polygons are the `xy` entries of that JSON. The progress notebook does not read Mexico City or Palu.

Correlation loss for the progress report is [src/correlation_loss.ipynb](src/correlation_loss.ipynb). It copies the preprocessing below and scores luminance correlation on 13 practice pairs. Each score writes under its own folder, with `progress/` for the midpoint run and `final/` for the later full pass:

```
outputs/correlation_loss/progress/tables/
outputs/correlation_loss/progress/figures/
outputs/correlation_loss/final/tables/
outputs/correlation_loss/final/figures/
outputs/change_vector/...
outputs/morphology/...
outputs/lab_histogram/...
outputs/texture_contrast/...
outputs/fusion/final/...
```

`tables/` holds the CSV and the metrics file. `figures/` holds the PNGs. The CSV columns stay `image_id`, `building_id`, `score`, and `n_pixels`, so the tables can be joined after every score has finished. This notebook fills `outputs/correlation_loss/progress/` only.

OpenCV is not in the system Python. The environment is the same stack as Lab 1. Run the notebook from this folder or from `src/`:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute src/correlation_loss.ipynb --inplace
```

If the data are not at `../Dataset/train`, set `XVIEW2_ROOT` to the `train` directory. For Colab, zip only those 13 pairs so the archive still contains `train/images` and `train/labels`, unzip it, and point `XVIEW2_ROOT` at that `train` folder. The full training set stays on the local disk.

The notebook already contains an executed run, so opening it shows the figures and the partial table. Re-running it needs Jupyter as well as the packages in `requirements.txt`.

Each score stays independent and follows the same contract. The score reads a pre-event image, a post-event image, and the post-disaster polygons. It does not read another score and it does not read the damage label. It writes a CSV with `image_id`, `building_id`, `score`, and `n_pixels`. A higher score means a larger apparent change.

Pelin Durukan writes the preprocessing once, and every other script copies it. The post-event image is histogram-matched to the pre-event image, channel by channel, with 256 bins. Very dark, smooth pixels are left out of that match. Around a polygon whose short side is at least 12 pixels, the post-event patch is shifted by normalized cross-correlation over at most 4 pixels. The polygon is then eroded by one pixel. If fewer than 40 pixels remain, the original polygon is used. She also writes the 70/30 practice split: image number modulo 10, with 0–6 in the fit split and 7–9 in the holdout.

The five scripts do not call each other. Ali Yılmaz combines the five tables only after they are all finished.

## Before the progress report

The run can be incomplete. The group agrees on one shared subset of practice pairs, large enough to grade buildings and to compute a destroyed-versus-rest F1 and an undamaged false-positive rate. It does not have to cover every practice event. Each person reports their own score. A score that is not ready yet stays blank. The caption says the run is incomplete and is not the evaluation. The scores are not weighted together yet. Mexico City and Palu stay out of this table.

- **Ali Yılmaz — correlation loss.** On luminance inside the polygon, the score is one minus the Pearson correlation. Because the correlation lies between −1 and 1, the score lies between 0 and 2 and is not clamped to the range from 0 to 1. Two flat patches score 0. A flat patch paired with a textured one scores 2. He runs it on the shared subset and writes that section. He does not weight the other scores yet.
- **Furkan Gönen — change-vector magnitude.** The score is the mean of the color-difference length inside the polygon. He runs it on the shared subset and draws those pairs: pre-event, post-event, and the magnitude, with polygons on top. One pair should show a real loss. One should show an intact building with a shadow or a shift.
- **Mehmet Ali Bağıbala — texture contrast.** Luminance inside the polygon, 8 gray levels, GLCM contrast at one pixel, averaged over 0°, 45°, 90°, and 135°. The score is the absolute difference of that contrast. He runs it on the shared subset, counts buildings under 40 pixels, and checks the square-meter area formula on two images from the subset.
- **Pelin Durukan — Lab histogram distance and the shared preprocessing.** Chi-square distance between 8-bin histograms of the `a` and `b` channels inside the polygon. Lightness is left out. She also writes the preprocessing steps and the practice split. She runs the histogram score on the shared subset and reports that number. She does not fit weights.
- **Zeynep Okkıran — morphological profile.** White and black disk top-hats at radii 1, 2, and 4 pixels, and white and black area top-hats at 16 pixels. The white area top-hat is the image minus its area opening, and the black area top-hat is the area closing minus the image. The score is the mean of the absolute differences of those residuals inside the polygon. The sizes stay fixed. She runs it on the shared subset and shows it on the same pairs.

## Between the progress report and the final report

The partial numbers are kept. Each person finishes a table for all nine events. The tables meet only after that.

- **Ali Yılmaz** finishes the correlation table. Then he runs the combination: z-scores on the fit split, nonnegative weights that sum to 1, three grade cuts, the holdout check, and one pass on Mexico City and Palu. A Gaussian Bayes rule on the same five scores, fit only on the practice fit split and held to the same false-positive cap, is a comparison column. The weighted sum is the result. If that result misses the minimum, one published network may be run once, unchanged, in a separate column. The correlation is not adjusted after the evaluation is seen.
- **Furkan Gönen** finishes the magnitude table. After all five tables exist, he draws one practice destruction, one practice false-alarm case, one Mexico City pair, and one Palu pair, with all five scores on the same footprints. Those evaluation drawings are illustrations. They are not a reason to change a formula.
- **Mehmet Ali Bağıbala** finishes the texture table. Once the grades exist, he computes the area from the original un-eroded polygons. Predicted area is the polygon area of buildings graded major or destroyed. The reference is the same area on the original ground-truth polygons of those two classes. The required error is on Palu and on the pooled evaluation. Mexico City is shown and is not required. The area intersection-over-union (IoU) of the two polygon sets is shown the same way.
- **Pelin Durukan** finishes the histogram table and writes where a change of color separated the classes and where it did not. The shared preprocessing stays hers. The weights stay with Ali.
- **Zeynep Okkıran** finishes the morphology table and writes what the fixed sizes caught and what they missed. She does not search for a new size on the evaluation.

The minimum is a destroyed-versus-rest F1 of 0.35, a major-or-destroyed F1 of 0.40, a Mexico City false-positive rate of at most 0.25, and an area error of at most 50% on Palu and on the pooled set. The aim is 0.60, 0.60, a false-positive rate of at most 0.10, an area error of at most 30% on those sets, and a four-class macro F1 of 0.35. The macro figure has no minimum.
