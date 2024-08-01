---
title: "Coding Bugs with ChatGPT"
read_time: true
date: 2024-08-01T00:00:00+05:30
tags:
  - ai
  - javascript
---

Recently I was tinkering around with creating a [Chrome extension](https://developer.chrome.com/docs/extensions). Not having a lot of experience with this, I did what the cool kids these days are doing: I decided to use [ChatGPT](https://openai.com/chatgpt/) for helping me write code. And don’t get me wrong, ChatGPT (or other AI tools for that matter) does a decent job of helping you create things and spot bugs. But it’s a gun that can easily backfire if you don’t know what you’re doing with it. This blog is going to be about one such incident that happened to me when working on this project.

## Adding Toggle Functionality to the Chrome Extension

The feature I wanted to work on was adding an on/off switch for the Chrome extension. This is a common feature in a lot of extensions where, when you click the extension icon from the menu, you see a button that lets you toggle the state of the extension. I couldn’t find any step-by-step guide on how to do this online, and I was feeling a little lazy to figure it out on my own by reading docs, so I decided to take the help of our friendly neighborhood AI.

Before I show you the code, let me give you some [basic idea](https://itnext.io/all-youll-ever-need-to-know-about-chrome-extensions-ceede9c28836) about how Chrome extensions work, in case you’re unfamiliar. There are three main files you need to know about:

- popup: This has the code related to the window you see when you click the extension icon from the extensions tab in the top right corner of your browser window.
- content: This is the stuff that gets rendered on a particular browser page. For example, the extension could add elements, change the color of things, or anything else.
- background: This is the stuff that keeps running in the background. It has nothing to do with the page you are on or the user clicking the extension to open the popup. This is very useful to implement cross-page stuff.

So here’s the code ChatGPT gave me for implementing the toggle functionality:

First of all, in the `popup.js` to change the state of the button shown in the `popup.html` when it is clicked:

```javascript
document.addEventListener("DOMContentLoaded", function () {
  const toggleButton = document.getElementById("toggleButton");

  chrome.runtime.sendMessage({ action: "getStatus" }, (response) => {
    if (response && typeof response.isEnabled !== "undefined") {
      updateButtonText(response.isEnabled);
    } else {
      console.error("Invalid response:", response);
    }
  });

  toggleButton.addEventListener("click", () => {
    chrome.runtime.sendMessage({ action: "toggle" }, (response) => {
      if (response && typeof response.isEnabled !== "undefined") {
        updateButtonText(response.isEnabled);
      } else {
        console.error("Invalid response:", response);
      }
    });
  });

  function updateButtonText(isEnabled) {
    toggleButton.textContent = isEnabled ? "Turn Off" : "Turn On";
    toggleButton.style.backgroundColor = isEnabled ? "#f44336" : "#4caf50";
  }
});
```

Then in the `background.js`, to have listeners that store the state in the Chrome local storage and send the latest state back when asked for:

```javascript
// background.js
let isEnabled = true;

chrome.runtime.onInstalled.addListener(() => {
  console.log("Extension installed");
  chrome.storage.local.set({ isEnabled: true }, () => {
    if (chrome.runtime.lastError) {
      console.error("Error setting initial state:", chrome.runtime.lastError);
    }
  });
});

chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === "toggle") {
    isEnabled = !isEnabled;
    chrome.storage.local.set({ isEnabled }, () => {
      sendResponse({ isEnabled });
    });
    return true;
  } else if (message.action === "getStatus") {
    sendResponse({ isEnabled });
    return true;
  }
});
```

And finally, in the `content.js`, I added this function which would send a message to the Chrome runtime to check if the extension is enabled or not and run a callback function if enabled:

```javascript
function checkOnOff(callback) {
  chrome.runtime.sendMessage({ action: "getStatus" }, (response) => {
    if (response && typeof response.isEnabled !== "undefined") {
      callback(response.isEnabled); // Pass the state back to the caller via the callback
    } else {
      console.error("Invalid response:", response);
      callback(null);
    }
  });
}
```

Now on paper, this looks good, right? The code makes sense when you read it. And when I ran it as well, it worked perfectly.

## The Hidden Bug in ChatGPT’s Code

Only a few days after I had merged this code, I noticed a weird bug. I would turn the extension off, but after some time, it would turn back on. And I wasn’t able to reproduce this accurately each time. I didn’t know what was causing it to turn on again: was it just after a fixed period of time, was it me clicking somewhere on the screen, or me changing tabs, or something else completely? No idea.

I knew it was this PR which introduced this bug (benefits of using version control for personal projects as well), so I went back. I decided I’m going to make the culprit fix their own mistake. I did have a slight idea of what might be going wrong, but I still decided to run it by our flawed expert. And what it caught was exactly along the lines of what I was thinking. If you look at the earlier code:

```javascript
// background.js
let isEnabled = true;

...

chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
		if (message.action === 'toggle') {
        isEnabled = !isEnabled;
        chrome.storage.local.set({ isEnabled }, () => {
            sendResponse({ isEnabled });
        });
        return true;
    } else if (message.action === 'getStatus') {
        sendResponse({ isEnabled });
        return true;
    }
});

```

you see that when the content script asks for status, we send the `isEnabled` variable instead of retrieving the set value from local storage. Now, this theoretically should still work since we change the value of the `isEnabled` variable when a toggle event occurs. I don’t understand the Chrome extension lifecycle enough to be sure, but what I suspect is that somehow this background script gets reset and so the value of `isEnabled` becomes `true` again, and that’s what gets sent as a response, causing the extension to toggle on again after some time/event.

The way to fix that is what I mentioned earlier. Returning the value of `isEnabled` after fetching it from the local storage, and that’s what ChatGPT suggested as well:

```javascript
// background.js
let isEnabled = true;

...

chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
		if (message.action === 'toggle') {
        isEnabled = !isEnabled;
        chrome.storage.local.set({ isEnabled }, () => {
            sendResponse({ isEnabled });
        });
        return true;
    } else if (message.action === 'getStatus') {
        chrome.storage.local.get('isEnabled', (result) => {
            sendResponse({ isEnabled: result.isEnabled });
        });
        return true;
    }
});

```

And after merging this, I had no problems!

## Lessons Learned Using AI for Coding

It always amuses me how these AI coding assistants give you code which has bugs but also have the capability to debug that very same code when given in a different prompt. No doubt they are useful, especially when bootstrapping a project or feature. They’re great at giving you boilerplate code. But the lesson I’ve learned is to treat it just like that - code to help get you started. Make sure you type that code on your own instead of just copying and pasting it. Because when copying and pasting, like I did here, everything makes sense on the screen, but when you put it in your codebase thinking it should work, it actually might not.

Instead, when you type it out, I think you’re much more aware of what you’re doing, and it’s much harder to fall into the trap of adding bug-filled AI code to your project.

Thanks for reading! I hope this was a fun read, and if you enjoyed reading it or learned something from it, consider subscribing to my newsletter so you get notified every time I post something!

{{< rawhtml >}}

<iframe
scrolling="no"
style="width:100%!important;height:220px;border:1px #ccc solid !important"
src="https://buttondown.email/arsh?as_embed=true"
></iframe><br /><br />
{{< /rawhtml >}}
