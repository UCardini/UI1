All Design documentation are in the Docs_Design_Drawings.pdf

Project 01 - Smart floor tile
Description

The smart floor tile is a device that allows families to keep track of ambient information as well as their individual personal health information all while being the centerpiece of the users home.
How to use the demo

There are 3 modes:

    Standby
    Interactive 1
    Interactive 2

Standby mode simulates what the floor tile will look like displaying it's ambient information. This is whatever the user decides to put on this mode using the Interactive 2 widgets.

Interactive 1 mode simulates one of 4 users stepping on the floor tile activating the weighing functionality of this product. To simulate this there is a semitransparent layer placed over the floortile where the user would probably stand in order to weigh themselves. You can select which user you would like to simulate by clicking and holding their name for 3+ seconds. This simulates that user walking up to the tile and attempting to read their current weight information. After 3 seconds or longer you can let go, simulating the user you have selected stepping off the device and reading their results the information will remain for several seconds before disappearing.

Interactive 2 mode simulates a user engaging the physical switch located next to the tile or through the companion app enabling them to control various options and the display of widgets on their floor tile as it's in standby mode. The controls are simple you can drag widgets from the top of them to place them where you like as well as delete and add new ones from the hamburger menu in the top right. On the bottom right of widgets you can resize them to fit within the 8x8 grid on the floor tile. You can also adjust and add/remove entrys from the todo list from this mode aswell.

This project was implemented using widgets for their modern athetics and ease of use by the end user. Most people have used widgets before from other applications and this should carry over when they're using the tile. Code structure separates the widgets into separate files and the usage modes are separated by pages. This was for making debugging and development easier.

AI was used to help troubleshoot the widgets and understand RSS feeds information.
