---
layout: post
title: "Android WebView Handles Key Events"
date: 2021-12-14 08:50:00 -0000
categories: Android WebView
---

## Introduction

This post will show you how to handle key events in WebView with JavaScripts.

## Enable JavaScript for WebView

Call `WebSettings: setJavaScriptEnabled` to enable JavaScript:

```
	WebSettings webSettings = webView.getSettings();
	webSettings.setJavaScriptEnabled(true);
```

## Dispatch Key Event to WebView

Override `onKeyDown` method of the activity holds this WebView:

```
    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
    	if (keyCode == KeyEvent.KEYCODE_BACK) {
    		this.webView.evaluateJavascript("javascript:onBackKeyPress()", new ValueCallback<String>() {
                @Override
                public void onReceiveValue(String value) {
                	// TODO: handle return value from JS code here.
                }
            });
        }

        return super.onKeyDown(keyCode, event);
    }

```
The `javascript:onBackKeyPress()` is JavaScript code, this is implemented in the HTML5 files. If the html file loaded by this WebView has this JavaScript function, it will be called.



