# knapsack problem:



Prerequisite to run the program:
1. java 1.8 .
2. docker 18 or latest
3. maven 3.6.x


Steps to launch the solution program:

1. download mongo.yml and kafka.yml file.
5. execute 'docker-compose -f mongo.yml' and 'docker-compose -f kafka.yml'  commands on terminal.
6. download project zip files(maersk-knapsack.zip, knapsack-solution.zip).
7. after extracting zip files
   
   ->Open terminal and navigate to project folder(maersk-knapsack) and run command 'mvn spring-boot:run'.
   ->Open terminal and navigate to project folder(knapsack-solution) and run command 'mvn spring-boot:run'.
   
   
 Solution architecture :
  I have implemented 2 microservices.
  1.** Maersk-knapsack microsservice:**  
  Which has exposed 2 end points  one for submit knapsack problem using (http://localhost:6543/knapsack) end point.
  it is a post type request.
  
  
    request paylaod sample:{"problem": {"capacity": 60, "weights": [10, 20, 33], "values": [10, 3, 30]}}
    
    
    As soon as Maersk-knapsack service  receives the request, it insert  request payload to mongodb databse  after updating status  'submitted' and     sumbitted time.
    
    
    mongodb generate unqiue id for each document which I am using as task id, maerk-service send message to kafka broker and return the response as below.
   
   
   
   response format:
    {
    "task": "60d0550bfe2bd64354931712",
    "status": "submitted",
    "timestamps": {
        "submitted": 1624265995932,
        "started": null,
        "completed": null
    },
    "problem": {
        "capacity": 60,
        "weights": [
            10,
            20,
            33
        ],
        "values": [
            10,
            3,
            30
        ]
    },
    "solution": {
        "packed_items": null,
        "total_value": null
    }
}
  
    
    
  
  2. Second endpoint is to check problem solution and its current execution status using ( localhost:6543/knapsack/{task_id} )
       
     request : http://localhost:6543/knapsack/60d0550bfe2bd64354931712
     
     
     maersk-knapsack services fetch information from mongodb using unique {task_id} and return below  response if it is avaialable:
     
     
     response:
   {
    "task": "60d0550bfe2bd64354931712",
    "status": "completed",
    "timestamps": {
        "submitted": 1624265995932,
        "started": 1624265996302,
        "completed": 1624265996466
    },
    "problem": {
        "capacity": 60,
        "weights": [
            10,
            20,
            33
        ],
        "values": [
            10,
            3,
            30
        ]
    },
    "solution": {
        "packed_items": [
            0,
            2
        ],
        "total_value": 40
    }
}

2. Knapsack-solution service:

 it keep  reqeusting new message from kafka broker  each 100 ms interval using kafka consumer interface. 
 As soon as it receivew message from kafka broker. It update the status to 'started' and  start time  of Problem and update the record in mognodb.
  Then, it calls knapsack alogrithm  and once algorith returns the result , it append the result to Problem solution and set 
 status to 'completed' and completion time 'current time' at that moment and update the record in mongodb.
   
