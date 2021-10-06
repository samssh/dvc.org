# DVCLive with DVC

Even though DVCLive does not require DVC, they can integrate in several useful
ways:

- The [_outputs_](#outputs) DVCLive produces can be fed as
  `dvc plots`/`dvc metrics`, making it easier to add metrics logging to DVC
  <abbr>stages</abbr>. Those same outputs can be visualized in
  [_DVC Studio_](#dvc-studio)
- You can monitor model performance in realtime with the
  [_HTML report_](#html-report) that DVCLive generates when used alongside DVC.
- DVCLive is also capable of generating [_checkpoint_](#checkpoints) signal
  files used by DVC <abbr>experiments<abbr>.

## Setup

We will refer to a training script (`train.py`) already using `dvclive`:

```python
# train.py

import dvclive

for epoch in range(NUM_EPOCHS):
    train_model(...)
    metrics = evaluate_model(...)

    for metric_name, value in metrics.items():
        dvclive.log(metric_name, value)

    dvclive.next_step()
```

Let's use `dvc stage add` to create a stage to wrap this code (don't forget to
`dvc init` first):

```dvc
$ dvc stage add -n train --live training_metrics
                -d train.py python train.py
```

`dvc.yaml` will contain a new `train` stage with the [`DVCLive configuration`]
(in the `live` field):

```yaml
stages:
  train:
    cmd: python train.py
    deps:
      - train.py
    live:
      training_metrics:
        summary: true
        html: true
```

The value passed to `--live` (`training_metrics`) became the directory `path`
for DVCLive to write logs in, and DVC will now
[track](/doc/use-cases/versioning-data-and-model-files) it. Other supported
command options for the DVC integration:

- `--live-no-cache <path>` - specify a DVCLive log directory `path` but don't
  tracked it with DVC. Useful if you prefer to track it with Git.
- `--live-no-summary` - deactivates
  [summary](/doc/dvclive/get-started#metrics-summary) generation.
- `--live-no-html` - deactivates [HTML report](#html-report) generation.

> Note that these are convenience CLI options. You can still use
> `dvclive.init()` manually, which will override any options sent to
> `dvc stage add`. Just be careful to match the `--live` value (CLI) and `path`
> argument (code). Also, note that summary files are never tracked by DVC
> automatically.

Run the training with `dvc repro`:

```dvc
$ dvc repro train
```

## Outputs

After that's finished, you should see the following content in the project:

```dvc
$ ls
dvc.lock  training_metrics       train.py
dvc.yaml  training_metrics.json
```

The `.tsv` files generated under `training_metrics` can be visualized with
`dvc plots`.

In addition, `training_metrics.json` can be used by `dvc metrics` and visualized
with `dvc exp show`/`dvc exp diff`.

### DVC Studio

[DVC Studio](/doc/studio) will automatically parse the outputs generated by
DVCLive, allowing to
[compare and visualize](/doc/studio/user-guide/visualize-experiments)
experiments using DVCLive in DVC Studio.

![](/img/dvclive-studio-plots.png)

### HTML report

In addition to the
[outputs described in the Quickstart](/doc/dvclive/get-started#outputs), DVC
generates an _HTML report_.

If you open `training_metrics_dvc_plots/index.html` in a browser, you'll see a
plot for metrics automatically updated during the model training!

![](/img/dvclive-html.gif)

### Checkpoints

When used alongside DVC, DVCLive can create _checkpoint_ signal files used by
DVC <abbr>experiments<abbr>.

This will save the metrics, plots, models, etc. associated to each
[`step`](/doc/dvclive/api-reference/get_step).

You can learn more about how to use them in the
[Checkpoints User Guide](/docs/user-guide/experiment-management/checkpoints) and
in this example
[repository](https://github.com/iterative/dvc-checkpoints-mnist).

[`dvclive configuration`]: /doc/dvclive/api-reference/init#parameters