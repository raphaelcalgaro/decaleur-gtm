# Decaleur and Send to secondLayer - GTM Custom Tag Templates

These tag templates help avoid the unassigned traffic source by making sure no custom event is fired before the page_view or any specified events.

## Description
Decaleur is a custom tag template for Google Tag Manager (GTM) designed to manage the execution order of events, ensuring that the critical specified event is triggered first.

## WHY?
When the first event of a new session is not a page_view, the __traffic source associated with this is set as UNASSIGNED__. This happens a lot in ecommerce sites for pages with product listings or a product description as the * *view_item_list* * or * *view_item* * event can be triggered before the * *page_view* *. So I created a pair of Tag templates that 
 - Creates a flag, by default set to false to indicate that the page_view was fired or not. This flag is used to fire the events that would normally fire
 - The __Send to secondLayer__ tag template is used to send events to the 2nd layer in the dataLayer waiting for the page view to fire.
 - Once the page view fires, the Décaleur changes the flag to the value true and fires every blocked event's name and parameters that were sent to the secondary level in the data layer with the Send to secondLayer tag.

## Key Features:
1. Prioritization of page_view:
   - Prevents subsequent events from firing until the page_view event has been recorded.

2. Queue for Deferred Events:
   - Events triggered before page_view are temporarily stored in a secondary queue named dataSecondLayer.

3. Queue Flush after page_view:
   - Once the page_view event is detected, all deferred events in dataSecondLayer are fired with their respective parameters.

4. Parameter Management and Cleanup:
   - Parameters used in deferred events are automatically cleared when sent to backlog and after usage once the page_view has fired, ensuring they are not reused unintentionally.

5. Seamless Integration with GTM:
   - Leverages GTM’s native APIs like createQueue, copyFromDataLayer, and setInWindow to ensure compatibility.

## Usage
1. Import the Templates into GTM:
   - Import the page_view_first.tpl and the Send to secondLayer.tpl files into your GTM container.

2. Create the Decaleur tag
   - Go in the tag page
   - Click New
   - Chose Decaleur
   - Enter a name for the tag
   - Enter the name of the event that triggers your page_view
   - Add the "Initialization - All Pages" trigger
   - Add the trigger that also fires on your page_view events.

3. Create a JavaScript Variable for the flag pageViewDetected and a Data Layer Variable
   3.1 JavaScript Varaible
     - Go to the variables page
     - Click "new" in the "User-Defined Variables" section
     - Choose the JavaScript Variable
     - Enter a name. I used "VarGlobale - window.pageViewDetected"
     - Enter the Global Cariable name as "window.pageViewDetected"
   3.2 Data Layer Variable
     - Create a new dataLayer variable Where the Data Layer Variable Name is "eventName"

4. For each of the events that might be causing a problem with Unassigned traffic sources by firing before the page_view, edit the trigger, create a pre-page_view event copy and create a delayed event firing trigger
   4.1 edit the main trigger
       - Add a condition under "This trigger fires on"
       - the variable name is the one defined in step 3.1
       - Set is operator as "contains" and the value as true.
       - Rename it so you uderstand that this tag will now only fire after the page_view.
   4.2 Then create a copy of each 4.1 trigger for before the page_view is fired.
       - On those set the javaScript variable defined at the last step as false
       - Add these triggers to the problematic events but be carefull that all Event Settings Variable are still valid when the delayed event fires after the page_view
   4.3 Create a copy of every 4.1 triggers
       - in thess new copies change the javaScript variable we just added by the Data Layer Variable we created in step 3.2
       - Set the Data Layer Variable's value to the original 4.1 trigger's event name
       - Change the new trigger's Event name to "firingDelayedEvent"

7. Create a "Send to secondLayer" Tag for each of the events that might be causing a problem with Unassigned traffic sources
   - Go to the tag page and click new
   - Select the "Send to secondLayer" Tag
   - Define the event name that will eventually need to be sent to GA4
   - If you also have event parameters that needs to be defined check the box and enter the keys and values
     - The tag does not currently support the User Defined Variables
     - You can use the variables to fill the parameters and their values
   - Add the relevant pre-page_view trigger(s) created in the last step

## Example Scenario:
1. Before * *page_view* *:
   - Events such as * *view_item_list* * and *  *view_item* * are stored in dataSecondLayer.

2. After page_view:
   - The Decaleur tag flushes the queue, fires the stored events, and clears the associated parameters.

## Benefits:
   - Ensures data consistency by prioritizing critical events like page_view.
   - Maintains parameter integrity by clearing used parameters after events are triggered.
   - Help ensure that the controlled events are fired ater the page_view so the traffic source is not set as unassigned.
