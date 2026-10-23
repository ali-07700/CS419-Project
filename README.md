# CS 419 — Post-earthquake change detection

Group: Ali Yılmaz, Furkan Gönen, Mehmet Ali Bağıbala, Pelin Durukan, Zeynep Okkıran.

Paired satellite images are compared inside known building polygons. Five scores are computed from the same histogram-matched pairs, then combined with weights chosen on non-earthquake events. Mexico City and Palu are scored once, after those weights are frozen.

The images are the xView2 challenge training set. They are not stored in this repository. Place that download next to this folder, as `../Dataset`, and do not commit it.

Each score is one Python script. The script reads a pre-event image, a post-event image, and the post-disaster polygons. It does not read another score and it does not read the damage label. It writes a CSV with `image_id`, `building_id`, `score`, and `n_pixels`. A higher score means a larger apparent change. Before the score, each color channel of the post-event image is histogram-matched to the same channel of the pre-event image, with 256 bins.

## Before the progress report

The run can be incomplete. The group agrees on one shared subset of practice pairs, large enough to grade buildings and to compute a destroyed-versus-rest F1 and an undamaged false-positive rate. It does not have to cover every practice event. Scores that are not ready yet stay blank in the table. The caption says the run is incomplete and is not the evaluation. Mexico City and Palu stay out of this table.

- **Ali Yılmaz — correlation loss.** On luminance inside the polygon, the score is one minus the Pearson correlation. Two flat patches score 0. A flat patch paired with a textured one scores 2. He also writes the practice split file: image number modulo 10, values 0–6 in the fit split and 7–9 in the holdout. For the progress report he runs the score on the shared subset and writes the correlation section.
- **Furkan Gönen — change-vector magnitude.** The score is the mean of the color-difference length inside the polygon. He runs it on the shared subset and draws those pairs: pre-event, post-event, and the magnitude, with polygons on top. One pair should show a real loss. One should show an intact building with a shadow or a shift.
- **Mehmet Ali Bağıbala — top-hat difference.** White top-hat of the luminance with a disk of radius 2 pixels. The score is the absolute difference of the mean top-hat inside the polygon. The radius stays fixed. He runs it on the shared subset and shows the top-hat on the same pairs.
- **Pelin Durukan — Lab histogram distance.** Chi-square distance between 8-bin histograms of the `a` and `b` channels inside the polygon. Lightness is left out. She also starts the combine script and tests it on made-up tables. On the scores that exist, she builds the partial table. A trial weight on the subset may be shown if at least two scores exist. It is labeled a trial.
- **Zeynep Okkıran — texture contrast.** Luminance inside the polygon, 8 gray levels, GLCM contrast at one pixel, averaged over 0° and 90°. The score is the absolute difference of that contrast. She runs it on the shared subset, counts buildings under 40 pixels, and checks the square-meter area formula on two images from the subset.

## Between the progress report and the final report

The partial table is kept. The remaining work is the full pass, the frozen weights, and one evaluation.

- **Ali Yılmaz** runs correlation on all nine events and checks that the other tables use the same building ids. He drafts the dataset and split section. The correlation is not adjusted after the evaluation scores are seen.
- **Furkan Gönen** finishes the magnitude table. After all five tables exist, he draws one practice destruction, one practice false-alarm case, one Mexico City pair, and one Palu pair, with all five scores on the same footprints. Those evaluation drawings are illustrations. They are not a reason to change a formula.
- **Mehmet Ali Bağıbala** finishes the top-hat table. After the weights are frozen, he reports the undamaged false-positive rate of the fused score on each practice event, with attention to Midwest flooding, Guatemala, the Southern California fire, and Hurricane Matthew.
- **Pelin Durukan** finishes the histogram table and runs the combine script: z-scores on the fit split, nonnegative weights that sum to 1, three grade cuts, the holdout check, and one pass on Mexico City and Palu. If the fused score misses the minimum, one published network may be run once, unchanged, in a separate column.
- **Zeynep Okkıran** finishes the texture table. She applies the area function to the fused grades on Mexico City and Palu. Predicted area is the polygon area of buildings graded major or destroyed. The reference is the same area on the ground-truth polygons of those two classes.

The minimum on that evaluation is a destroyed-versus-rest F1 of 0.40, a major-or-destroyed F1 of 0.45, a Mexico City false-positive rate of at most 0.25, and an area error of at most 50%. The aim is 0.60, 0.60, a false-positive rate of at most 0.10, an area error of at most 30%, and a four-class macro F1 of 0.40. The macro figure has no minimum.
