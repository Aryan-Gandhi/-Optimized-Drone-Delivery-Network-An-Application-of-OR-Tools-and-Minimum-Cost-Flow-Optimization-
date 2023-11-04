# Google-Hash-Code-Drone-Delivery
Optimizing for minimal delivery distance and maximum load utilization


The competition instructions are present in the hashcode-drone-delivery directory.

My notebook is heavily inspired by Alex Bader's solution - https://www.kaggle.com/code/spacelx/2020-hc-dd-2nd-place-solution-w-or-tools/notebook


GOOGLE HASH CODE DRONE DELIVERY PROBLEM

Introduction
This is my solution to the Hash Code drone delivery problem, using optimization and routing routines from the Google OR-Tools package. Let me know what you think and feel free to leave an upvote if you like this kernel! Please comment if you have any ideas on how to improve this code - there surely are lots of improvements possible.
Credits for data extraction: Application of Google OR-Tools
The general process of this solution is quite straightforward:
We determine where each product unit is to be delivered from to minimize total delivery distance. This does not make use of redistributing units between warehouses, we simply look at where products are now and where they need to go and attempt to deliver them on the most direct route possible. Certain products are only stored in one or two warehouses and may need to be delivered across the entire map.
For each warehouse separately, we look at all deliveries which need to be executed from there and combine them into single delivery routes as efficiently as possible to maximize the load utilization and minimize the travel distance of each route. We group orders by their "difficulty" (i.e., total weight and distance) to get a higher score, aka minimize the overall waiting time for the completion of an order.
Combining all single routes from all warehouses, we attempt to schedule each drone such that the distance between the end of a single route and the start of the next one are as close together as possible (this is the most computing-intensive part). We do this in steps, such that "easy/fast" orders are completed first for a higher score.


Optimize product distribution
Now the first real step toward a solution is to figure out where products are needed and where they are currently located. We look at this for each product separately, determining the current storage location (source) for all available units and all locations where this product needs to be delivered to (sink). From this information we can figure out the optimal way of distributing this product to all customers while minimizing the total distance which has to be covered to satisfy all needs. This is done by constructing a graph connecting all sources and sinks, where the cost of each connection is equal to the distance between the adjoining source (warehouse) and sink (customer). A solution is found by performing a minimum-cost flow optimization and the results for each product are saved in a combined dataframe.


Define "weight-distance"
Weight-distance indicates the "difficulty" of an order, i.e. the amount of resources needed for its fulfillment. We define it as the sum of the product weight units * distance units for each product which is to be delivered in order to fully ship this order.
For example, order A includes product 1 with weight 50 which needs to be shipped over a distance of 10 distance units (50 * 10 = 500 wd units) and product 2 with weigth 100 which needs to be shipped over a distance of 150 distance units (100 * 150 = 15000 wd units) - a total WD score of 15500.
We use these scores to
bundle products for easy-to-finish orders together on single delivery flights (routes)
give these routes priority when assigning delivery schedules (combinations of routes)
in order to finish "easy/quick" orders first and longer ones later. In a real life problem this might not be the perfect way of optimizing, as we prioritize finishing certain orders before others for the cost of a somewhat less optimal (but still pretty good) resource utilization (drones might not be loaded as much as they could or might travel a bit further than absolutely necessary).



Create delivery routes
We now know all single deliveries which need to be executed and try to combine them into delivery routes where load utilization is maximum and travel distance is minimum. Essentially, we combine products with appropriate weights which need to be delivered from the same warehouse into the same region of the map into efficient delivery routes using Google OR-Tools' routing logic.
Depending on the number of single products needing to be delivered this can be somewhat inefficient and slow if we wanted to calculate in one go all routes for all products which are to be delivered from the same warehouse - we can hence form subgroups depending on the delivery location before attempting this optimization.
However, in order to obtain better scores (deliver orders as early as possible, rather than as efficiently as possible) we can also create routes from subgroups of all product deliveries from a warehouse but grouped by the weight-distance score introduced above.
Or simply a combination of the two.



Turn routes into schedules
We now have a stack of single delivery routes which optimize load utilization and delivery distance. These need to be distributed across all available drones while minimizing total travel distance / distribution time and prioritizing short/quick orders. Following the same scheme as in the previous step, we use Google OR-Tools' routing logic to determine a schedule for each drone. Note that we batch the available routes by weight-distance value, i.e. by priority. This does not only lead to better scores but also to shorter computation times, compared to trying to find a single optimal solution using all routes (where distance and total time are minimized).




Calculate score
Let's go through the commands, drone by drone, carefully tracking time as well as currently loaded weight. We make sure to note a timestamp for each delivery, and we create a list of inventory actions (loading / unloading) which we'll use in the next cell to track warehouse inventory.

