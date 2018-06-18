---
author: admin
comments: true
layout: post
slug: building-services-using-automator-workflows-in-snow-leopard-10-6
title: Building Services Using Automator Workflows in Snow Leopard (10.6)
wordpress\_id: 775
tags:
- automator
- mac
- services
---

In Snow Leopard (Mac OS X 10.6) the Automator tool has been drastically upgraded to support the creation of service workflows.  In simple terms, this means you can build automated chains of tasks that can be invoked in a context sensitive manner.  Not simple enough?  Using this tool, you can automate common actions you perform and the proper service will appear in the menu only when it is capable of being used.  You can even assign global hotkeys (via the Keyboard preference pane) to your service.  Let's take a look at a simple service workflow so you can see how it's done.

Open Automator (it's located in your /Applications folder).  You will be greeted by a sheet requesting a template for your workflow.  Choose service.

[![choose_your_template](/assets/media/2009/11/choose_your_template-300x278.png)](/assets/media/2009/11/choose_your_template.png)

You'll now be greeted by the main Automator window.  On the left you'll see a Library and actions you can perform.  On the right you'll see an empty pane where you drag actions or files to build your workflow.  Third party applications you have installed can expose additional available actions, so the list of available actions will vary for each user's computer.

[![main_window](/assets/media/2009/11/main_window-300x265.png)](/assets/media/2009/11/main_window.png)

As an example, we're going to build a service that allows you to select files in the Finder, automatically zip them, and attach them to a new email.  To begin, change the drop down in the upper right from "text" to "files or folders" and change "any application" to "Finder".  This means that the service we're constructing will accept files or folders from the Finder as input.  Next we need to add our first action.  To accomplish this type "create" in the search field in the upper left of the actions pane.  This will live filter the available actions so you can more easily locate the "Create Archive" action.  Once you've found it, drag and drop it to the right hand side.

After creating the zip archive we want to attach it to a mail message.  Search for "new mail message" and drag and drop it below the first action.  It will prompt you with a few dialogs (mostly for permissions to access your keychain), but when you're done you should have the workflow below.

[![prelim_workflow](/assets/media/2009/11/prelim_workflow-300x247.png)](/assets/media/2009/11/prelim_workflow.png)

To test you can click Run, but you'll receive a warning that states "This service will not receive input when run inside Automator.  To test this service within Automator, add the “Get Specified Finder Items” action to the beginning of your workflow. Remove or disable the action before running the workflow outside of Automator.".  Hit cancel (there's no point to running our service without input!) and add the "Get Specified Finder Items" action to the very top of your workflow.  You'll also need to add an item or two for testing.

[![get_specified](/assets/media/2009/11/get_specified-300x247.png)](/assets/media/2009/11/get_specified.png)

To test the entire service, click Run in the upper right!  You should see it create a zip file in the same directory as whatever files you specified, then open a new Mail message with your zip file attached.

If everything is working you can remove or disable the "Get Specified Finder Items" action and save your new service.  Whatever you name it will be how it appears in your Services menu, so try to give it a descriptive name (something like "Create Zip and Email").  Now head to the Finder, select a file or folder, and click the Finder menu to see the Services submenu.

[![services_menu](/assets/media/2009/11/services_menu-300x117.png)](/assets/media/2009/11/services_menu.png)

Success!

But what if you don't want to use that submenu?  OS X allows you to assign global shortcuts/hotkeys to any Service using the Keyboard preference pane in System Preferences.  Go to Keyboard Shortcuts within it, click the services option on the left, then find your "Create Zip and Email" option on the right.  To add a shortcut you'll need to double click on the righthand side of the Create Zip and Email row.  Once the text box appears, press the key combination you want to use to invoke this service.  Try to choose one that won't conflict with anything.

[![keyboard_assigned](/assets/media/2009/11/keyboard_assigned-300x271.png)](/assets/media/2009/11/keyboard_assigned.png)

Automator has a great deal of power, so try experimenting with various actions to build your own ideal workflow.  For example, I frequently parse SSL certificates in the course of my day, so I built this workflow to do it quickly and display the results in a text document in my editor of choice:

[![parse_cert](/assets/media/2009/11/parse_cert-290x300.png)](/assets/media/2009/11/parse_cert.png)

I also wanted a universal key combo to start the screensaver (which requires a password) so I could lock my computer quickly without using the mouse.

[![start_screensaver](/assets/media/2009/11/start_screensaver-291x300.png)](/assets/media/2009/11/start_screensaver.png)

If you've got other service ideas or want to show off one of you've written drop a comment!  It's always great to discover a better/more efficient way to accomplish a task.
