# Debugging Overlays
 
osu! framework provides in-picture debugging overlays, defined in `osu.Framework.Input.FrameworkActionContainer`. 

## Draw hierarchy overlay

**Control-F1** toggles an overlay that allows you to access the drawable hierarchy. Selecting an object on screen will grab the instance of the drawable and display relevant information.

![](https://cdn.discordapp.com/attachments/318886668889227266/538272068764958720/Screen_Shot_2019-01-25_at_5.10.57_PM.jpg)

The first time you click a drawable, you will be shown the the first container that is forward facing the user that was clicked, which means it is possible the element you want to get to is not accessible in this way. In order to gain access to the other drawables, you will have to traverse the hierarchy by clicking the "up one parent" button until you reach the desired depth.

![](https://cdn.discordapp.com/attachments/318886668889227266/538306806795993090/Screen_Shot_2019-01-25_at_7.28.11_PM.jpg)

Greyed out items in this list mean that there are additional children that are hidden from this list that you can expand by clicking on them. New transformations applied on a drawable will be represented in the form of yellow flashes appearing to the next of their respective items on the list.

While now we are able to see a lot of the drawables in this list, it does prove to be a rather large amount of clutter. To clear all items on the list except for a specific container and its children, simply double click on it.

![](https://cdn.discordapp.com/attachments/318886668889227266/538310347631755265/Screen_Shot_2019-01-25_at_7.28.11_PM.jpg)

All properties for each container can be accessed using the "view properties" button, or right clicking the drawable you want to access.

![](https://cdn.discordapp.com/attachments/318886668889227266/538311755491835904/penis.jpg)

## Frame statistics

* **Control-F11** toggles the frame statistics overlay. This displays the frame timings of the four main gameplay threads: Audio, Input, Update, and Draw. 

![](https://cdn.discordapp.com/attachments/318886668889227266/538264876561203200/Screen_Shot_2019-01-25_at_3.31.08_PM.png)

**Pressing Control-F11 again** will expand the overlay, displaying a graph of the activity on each thread. 

![](https://cdn.discordapp.com/attachments/318886668889227266/538266387014221824/Screen_Shot_2019-01-25_at_4.58.06_PM.png)

**Holding down the control key** in this state will expand the graph further, allowing you to peek at how many calls of each type are occuring within the timeframe.

![](https://cdn.discordapp.com/attachments/318886668889227266/538268818951241754/unknown-2_copy.png)

## Log Viewer

**Control-F10** toggles the log viewer, which displays events logged by `osu.Framework.Logging.Logger`. Invoking `Logger.Log` will log messages that are able to be displayed in this stream..




