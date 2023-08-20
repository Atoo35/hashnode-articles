---
title: "A better Gin/Gonic + Service Weaver approach"
seoTitle: "Better approach for creating microservices using service weaver"
datePublished: Sun Aug 20 2023 09:35:16 GMT+0000 (Coordinated Universal Time)
cuid: cllj95q8q000i09jp0og16iq3
slug: better-gingonic-service-weaver-approach
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1692524014906/40f70a97-aacb-4cbb-aa77-1e69a615216e.webp
tags: microservices, golang, google, gin-gonic, serviceweaver

---

Hola Amigos!!! Today we will be building on the previous article [Gin/Gonic + Service Weaver](https://atoo.hashnode.dev/gingonic-service-weaver) and implementing a better approach for using Service Weaver and structuring your project even better.

Here is the link to [GitHub Repository](https://github.com/Atoo35/gingonic-service-weaver/tree/service-infra). You can go ahead and get the code directly but I would recommend reading through the article once.

> Note: Make sure you are on the `service-infra` branch.

# Prerequisites

* GO installed in the system
    
* Basic understanding of GO
    
* Knowledge of REST APIs
    
* [Service Weaver](https://serviceweaver.dev/docs.html#installation) installed in your system
    
* Having read the previous [article](https://atoo.hashnode.dev/gingonic-service-weaver)
    

I would recommend reading through some of the Service Weaver docs before reading further since some implementation and deployment aspects are better explained in the docs.

# Let's Build it 🛠️

This is what the project structure would look like and I am sure you can already see it makes much more sense from a microservice point of view, although it is being written as a monolith (oooof!!!).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692520987390/ebb6dbe6-9159-4222-bc1b-d915ddb3dcb0.png align="center")

The `mock` and the `models` folder remains the same, no change there. We now divide the project into more logical pieces like the `taskservice` defines all the code related to the management of the task. I have introduced a new service called `notificationservice` which is just a dummy service in this project to demonstrate how services can be broken down and used across services in a typical service weaver project.

A fun part to be discovered later on is that we only expose the task service as an HTTP server and not the notification service, which indicates that we can have so-called "*internal*" services like in a Kubernetes orchestration.

1. Let's build out the `taskservice` first. Create a file `server.go` in the folder and copy paste the below code.
    
    ```go
    package taskservice
    
    import (
    	"context"
    
    	"github.com/Atoo35/gingonic-service-weaver/notificationservice"
    	"github.com/ServiceWeaver/weaver"
    	"github.com/gin-gonic/gin"
    )
    
    type Server struct {
    	weaver.Implements[weaver.Main]
    	taskapi weaver.Listener
    	handler *gin.Engine
    
    	notificationService weaver.Ref[notificationservice.Service]
    }
    
    func (s *Server) Init(ctx context.Context) error {
    	router := gin.Default()
    
    	tasksRoutes := router.Group("/api/tasks")
    	{
    		tasksRoutes.GET("/", s.GetTasks)
    		tasksRoutes.POST("/", s.CreateTask)
    		tasksRoutes.GET("/:id", s.GetTask)
    		tasksRoutes.PUT("/:id", s.UpdateTask)
    		tasksRoutes.DELETE("/:id", s.DeleteTask)
    	}
    	s.handler = router
    	return nil
    }
    
    func Serve(ctx context.Context, server *Server) error {
    	server.Logger(ctx).Info("Task API listening on ", "addr:", server.taskapi)
    	router := server.handler
    	return router.RunListener(server.taskapi)
    }
    ```
    
    So let's understand what is happening here in the file. First, we define a struct `Server` that implements the `Main` type of Service Weaver. This tells the framework that this is the entry point of the software.
    
    ```go
    type Server struct {
    	weaver.Implements[weaver.Main]
    	taskapi weaver.Listener
    	handler *gin.Engine
    
    	notificationService weaver.Ref[notificationservice.Service]
    }
    ```
    
    As before we also attach a Listener and a handler to deal with routing of the request. `notificationserivce` is referenced here to enable the handler get the service internally and use its functions.  
    The `Init` method is like any other `init` method in golang. It is run once before the other functions in the file are run. Here, we setup the routes and return the handler.
    
    Like before `Serve` function runs the server on the listener we created above i.e. `taskapi`
    
2. Next, we copy paste the below code in the `handler.go` file. This file is almost the same as the `api/handlers/task.go` file in the previous article.
    
    ```go
    package taskservice
    
    import (
    	"context"
    	"net/http"
    	"strconv"
    
    	"github.com/Atoo35/gingonic-service-weaver/mock"
    	"github.com/Atoo35/gingonic-service-weaver/models"
    	"github.com/gin-gonic/gin"
    )
    
    var ctx = context.Background()
    
    func (s *Server) GetTasks(gctx *gin.Context) {
    	tasks := mock.Tasks
    	if err := s.notificationService.Get().Send(ctx); err != nil {
    		s.Logger(ctx).Error("Failed to send notif")
    		gctx.AbortWithStatusJSON(http.StatusInternalServerError, gin.H{
    			"message": "Something went wrong while sending notification",
    		})
    	}
    	gctx.JSON(http.StatusAccepted, gin.H{
    		"tasks": tasks,
    	})
    }
    
    func (s *Server) GetTask(gctx *gin.Context) {
    	id := gctx.Param("id")
    	task := new(models.Task)
    	for _, value := range mock.Tasks {
    		if value.ID == id {
    			task = &value
    			break
    		}
    	}
    
    	if task.ID != "" {
    		gctx.JSON(http.StatusOK, gin.H{
    			"task": task,
    		})
    	} else {
    		gctx.JSON(http.StatusNotFound, gin.H{
    			"message": "Task not found",
    		})
    	}
    }
    
    func getTaskByID(id string) *models.Task {
    	task := new(models.Task)
    	for _, value := range mock.Tasks {
    		if value.ID == id {
    			task = &value
    			break
    		}
    	}
    	return task
    }
    
    func (s *Server) CreateTask(gctx *gin.Context) {
    	body := models.Task{}
    
    	if err := gctx.ShouldBindJSON(&body); err != nil {
    		gctx.AbortWithStatusJSON(http.StatusBadRequest, gin.H{
    			"message": "Bad body",
    		})
    		return
    	}
    
    	body.ID = strconv.Itoa(len(mock.Tasks) + 1)
    	alltasks := append(mock.Tasks, body)
    	gctx.JSON(http.StatusCreated, gin.H{
    		"tasks": alltasks,
    	})
    }
    
    func (s *Server) UpdateTask(gctx *gin.Context) {
    	id := gctx.Param("id")
    	task := getTaskByID(id)
    
    	if task.ID == "" {
    		gctx.AbortWithStatusJSON(http.StatusNotFound, gin.H{
    			"message": "Task not found",
    		})
    		return
    	}
    
    	body := models.Task{}
    
    	if err := gctx.ShouldBindJSON(&body); err != nil {
    		gctx.AbortWithStatusJSON(http.StatusBadRequest, gin.H{
    			"message": "Bad body",
    		})
    		return
    	}
    
    	var result []models.Task
    	for _, t := range mock.Tasks {
    		if t.ID == id {
    			result = append(result, body)
    		} else {
    			result = append(result, t)
    		}
    	}
    
    	gctx.JSON(http.StatusCreated, gin.H{
    		"tasks": result,
    	})
    }
    
    func (s *Server) DeleteTask(gctx *gin.Context) {
    	id := gctx.Param("id")
    	task := getTaskByID(id)
    
    	if task.ID == "" {
    		gctx.AbortWithStatusJSON(http.StatusNotFound, gin.H{
    			"message": "Task not found",
    		})
    		return
    	}
    
    	var result []models.Task
    	for _, t := range mock.Tasks {
    		if t.ID != id {
    			result = append(result, t)
    		}
    	}
    
    	gctx.JSON(http.StatusCreated, gin.H{
    		"tasks": result,
    	})
    }
    ```
    
    The only difference is the use of the `notificationservice` in the `GetTasks` function. Here we are simply getting the service and calling the `Send` function defined in the service. Nothing fancy at all.
    
    ```go
    if err := s.notificationService.Get().Send(ctx); err != nil {
    		s.Logger(ctx).Error("Failed to send notif")
    		gctx.AbortWithStatusJSON(http.StatusInternalServerError, gin.H{
    			"message": "Something went wrong while sending notification",
    		})
    	}
    ```
    
3. Let's create the `notificationservice/service.go` for better understanding as to what the `Send` function is actually doing.
    
    ```go
    package notificationservice
    
    import (
    	"context"
    
    	"github.com/ServiceWeaver/weaver"
    )
    
    type Service interface {
    	Send(ctx context.Context) error
    }
    
    type ServiceImplementation struct {
    	weaver.Implements[Service]
    }
    
    func (s *ServiceImplementation) Send(ctx context.Context) error {
    	s.Logger(ctx).Info("notification has been sent for x and y task")
    	return nil
    }
    ```
    
    Like a typical service weaver file we first define the interface `Service` which has the function `Send` and then define a struct `ServiceImplementation` that implements the service.  
    This method simply logs a line and does nothing else. As mentioned before, this is just to demonstrate how to have multiple services and run them together referencing across other services.
    
4. We are now pretty much done with everything except writing the `config.toml` and generating the auto-generated code. Simply open a terminal at the root directory and run the command
    
    ```bash
    weaver generate ./...
    ```
    
    This will generate all the `weaver_gen.go` files wherever required.
    
    > Note: I was facing some weird errors in another implementation (posting the article soon) while using the `weaver generate` command for each folder where required, and asked for help on the discord server and got to know that there is a topological order in which the generate command should be used, hence its better to use the command provided above since it automatically takes care of the ordering. Here is the response I got from one of the creators
    > 
    > ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692523147640/efff9d86-b948-41ab-bc1b-c80a79d9200b.png align="center")
    

1. The final file. Create the `config.toml` file and paste the following. Be sure to change the binary name.  
    
    ```bash
    [serviceweaver]
    binary = "./gingonic-service-weaver"
    
    [single]
    listeners.taskapi = {address="localhost:8080"}
    
    [multi]
    listeners.taskapi = {address="localhost:8080"}
    ```
    
    As before, we are telling weaver to use port `8080` for both single and multi-deploy. In multi deploy, replica sets would be generated and load balancing would be taken care of automatically using the round robin method.
    
    Run the `go build` command and then run either one of the below commands
    
    ```bash
    weaver single deploy config.toml
    ```
    
    ```bash
    weaver multi deploy config.toml
    ```
    
    To see what is happening in multi deploy, run `weaver multi status` and you would see something like the below
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692523852380/aaae103e-9e85-4779-b00b-ceda49c8d97d.png align="center")
    
    Notice that in Components you see 2 replica ids for both the services and there exists just one listener, i.e. the `taskapi` listener and the notification service is internal only.
    

That's it, you have successfully restructured the service weaver project and now it makes much more sense for the usage of the framework and also the project structure. Hope you found it useful and looking forward to comments.  

> Note: Something is coming up soon for the SurrealDB fans out there!!!

### **Support**

If you liked my article, consider supporting me with a coffee ☕️ or some crypto ( ₿, ⟠, etc)

Here is my public address `0x7935468Da117590bA75d8EfD180cC5594aeC1582`

[![Buy Me A Coffee](https://cdn.buymeacoffee.com/buttons/default-yellow.png align="left")](https://www.buymeacoffee.com/atoo)

### Let's Connect

[**Github**](https://github.com/Atoo35)

[**LinkedIn**](https://www.linkedin.com/in/atoo35)

[**Twitter**](https://twitter.com/atharva_35)

### Feedback

Let me know if I have missed something or provided the wrong info. It helps me keep genuine content and learn.