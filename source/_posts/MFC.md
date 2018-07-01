---
title: MFC
date: 2018-06-26 21:02:16
tags:
---
### Function call flow
```C++
CDialog::PreInitDialog()
CDialog::HandleInitDialog(WPARAM, LPARAM)
CDemoDlg::OnInitDialog()
CDialog::CheckAutoCenter()
CDialog::OnCmdMsg(nID, nCode, pExtra, pHandlerInfo);
CDialog::PreTranslateMessage(pMsg)
```

### Solve "Opened in another editor" in Resource View
Close "Resource.h" and all ".rc" file. Then shrink and re-expand the "rc" folder in Resource View.

