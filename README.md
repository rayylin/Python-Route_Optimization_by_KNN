# Route-Optimization-by-KNN
Case: a company want to build a new distribution center and would like to know which state is the best candidate.
We have the demand of each state and apply KNN to assign each state an appropriate distribution center
Have a look of the raw data 
![image](https://user-images.githubusercontent.com/58899897/194108411-759fbb19-d00c-4d3f-a734-7ed6a6c52f88.png)

Use Python to preprocess the raw data
If we use append, the dataframe would look like this 

![image](https://user-images.githubusercontent.com/58899897/194109457-888b42aa-f5c6-491d-9af6-d1900e604556.png)

To visualize the data, we need to do some preprocessing, and the result would look like this.

![image](https://user-images.githubusercontent.com/58899897/194109650-cd057ef4-650d-43d8-bef4-d024d24712cf.png)

Then we can visualize the data by Power BI

![image](https://user-images.githubusercontent.com/58899897/194108700-b096d0b6-a699-422a-abc3-2e8b3294686f.png)


We also have the latitude and longitude of each state, so we could use them to calculate the distance and perform optimization.
We use havershine package to calculate the distance.

This figure shows the result of iterarion 1

![image](https://user-images.githubusercontent.com/58899897/194111036-8693a322-a2a8-4cd2-bb1f-721ab0fbdb63.png)

The result with 5 distribution centers

![image](https://user-images.githubusercontent.com/58899897/194111312-1343bcfa-6b57-43b6-9aa4-73dfbd5c28c5.png)

The result with 6 distribution centers

![image](https://user-images.githubusercontent.com/58899897/194111244-4159b9ba-326d-4752-881e-f3d75464a5dd.png)






