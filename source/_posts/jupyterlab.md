---
title: jupyterlab升级后的数个插件问题
date: 2023-08-19 07:09:06
categories:
- Programming
- jupyter
---

第一，插件`@jupyterlab/notebook-extension:tracker`报错
```
[W 2023-08-12 15:11:31.242 LabApp] Failed validating settings (@jupyterlab/notebook-extension:tracker): Additional properties are not allowed ('experimentalDisableDocumentWideUndoRedo', 'numberCellsToRenderDirectly', 'observedBottomMargin', 'observedTopMargin', 'remainingTimeBeforeRescheduling', 'renderCellOnIdle' were unexpected)

    Failed validating 'additionalProperties' in schema:
        {'additionalProperties': False,
         'definitions': {'kernelStatusConfig': {'additionalProperties': False,
                                                'properties': {'showOnStatusBar': {'default': False,
                                                                                   'description': 'If '
                                                                                                  '`true`, '
                                                                                                  'the '
                                                                                                  'kernel '
                                                                                                  'status '
                                                                                                  'progression '
                                                                                                  'will '
                                                                                                  'be '
                                                                                                  'displayed '
                                                                                                  'in '
                                                                                                  'the '
                                                                                                  'status '
                                                                                                  'bar '
                                                                                                  'otherwise '
                                                                                                  'it '
                                                                                                  'will '
                                                                                                  'be '
                                                                                                  'in '
                                                                                                  'the '
                                                                                                  'toolbar.',
                                                                                   'title': 'Show '
                                                                                            'kernel '
                                                                                            'status '
                                                                                            'on '
                                                                                            'toolbar '
                                                                                            'or '
                                                                                            'status '
                                                                                            'bar.',
                                                                                   'type': 'boolean'},
                                                               'showProgress': {'default': True,
                                                                                'title': 'Show '
                                                                                         'execution '
                                                                                         'progress.',
                                                                                'type': 'boolean'}},
                                                'type': 'object'}},
         'description': 'Notebook settings.',
...
```

解决：从`~\.jupyter\lab\user-settings\@jupyterlab\notebook-extension\tracker.jupyterlab-settings`中删除以上选项。

第二，升级后部分Widgets无法正常显示，例如`tqdm`的进度条。

如果单单显示一个`Error display widgets`，那么首先要重新build插件。
```bash
jupyter lab build
```
如果提示node版本太低，就升级nodejs。

插件重新编译好后，应该可以进入js的阶段了。如果还是不能正常显示，显示一个框框让`Click to show javascript error`，那么可能需要升级`ipywidgets`的版本。
