---
title: "Simple API with Gin/Gonic and SurrealDB"
datePublished: Mon Jul 31 2023 13:44:58 GMT+0000 (Coordinated Universal Time)
cuid: clkqx9tkp000c09jwa68m3x66
slug: simple-api-with-gingonic-and-surrealdb
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1690811047695/6451c9e8-d916-4456-9f16-70b5c2876e7e.png
tags: go, opensource, sdk, gin-gonic, surrealdb

---

Hey everyone, here I am back with another article. This article is a tutorial on how to develop a REST API backend in GO making use of [gin/gonic](https://gin-gonic.com/) framework and [SurrealDB](https://surrealdb.com/) for the database

Here is the link to [GitHub Repository](https://github.com/Atoo35/gingonic-surrealdb). You can go ahead and get the code directly but I would recommend reading through the article once.

# Prerequisites

* GO installed in the system
    
* Basic understanding of GO
    
* Knowledge of REST APIs
    
* SurrealDB installed in your system
    

# Introduction

Hey there, in this tutorial we are going to be building a very rudimentary TODO REST API server, cause why not!! We would be using the gin/gonic framework to have an easy setup and use SurrealDB and its GO SDK to help ease the process of using it. Cheers to all the contributors of the SDK for making it smooth and doing the incredible heavy lifting for all of us (P.S.: I am one of them 😜). If you don't know what SurrealDB is, definitely recommend checking out their website, it's simply incredible and I loved using it as a database even for small side projects. Extremely light and easy to install and use.

# Let's Build it 🛠️

1. Create a folder for the project. I named my folder `gingonic-surrealdb`
    
2. Use the command `go mod init github.com/<your username>/<either folder name or any name you want>` . It's not mandatory to have the entire GitHub link as the module name, you can keep it anything you want. It's just a best practice in the golang industry.
    
3. Now create the following folder structure:
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690802632292/25f6995e-1144-4c5e-a934-a3cde49de820.png align="center")
    
      
    The `api` folder will contain code related to creating and handling the API calls. `database` folder is self-explanatory and would contain code to connect to SurrealDB. `models` represent the database models. Ignore the `tmp` folder since that is generated automatically by gin/gonic to log errors and build files
    
4. Let's start with the `database/database.go` file. Paste the below code into the file.  
    
    ```go
    package database
    
    import (
    	"log"
    
    	"github.com/surrealdb/surrealdb.go"
    )
    
    var DB *surrealdb.DB
    
    func Connect(connString, username, password string) {
    	var err error
    	DB, err = surrealdb.New(connString)
    	if err != nil {
    		log.Fatalf("Error connecting to database: %s", err)
    	}
    	_, err = DB.Signin(map[string]interface{}{
    		"user": username,
    		"pass": password,
    	})
    	if err != nil {
    		log.Fatalf("Error signing in: %s", err)
    	}
    
    	if _, err = DB.Use("test", "tasks"); err != nil {
    		log.Fatalf("Error using database: %s", err)
    	}
    
    	log.Printf("Connected to db with namespace %s and collection %s", "test", "tasks")
    }
    ```
    
    The `import` statement is used to fetch the package required into the file and make use of it. Here we are using the built-in `log` package to add some logs into the terminal and the `github.com/surrealdb/surrealdb.go` package which is the golang SDK for SurrealDB.
    
    > Note: You might get an error when you copy paste the code. Simply run `'go get install "github.com/surrealdb/surrealdb.go"` in the terminal to install the package.
    
    The `Connect` function accepts the connection string and the username and password used to connect to the database. This username and password is set when you start the SurrealDB instance by using the below command
    
    ```bash
    surreal start -u root -p root   
    ```
    
    We set the `namespace` and the database name in this function.
    
    ```go
    if _, err = DB.Use("test", "tasks"); err != nil {
    	log.Fatalf("Error using database: %s", err)
    }
    ```
    
    The namespace is `test` and the database name is `tasks`.
    

1. Let's now create the `models/task.go` file. This file is simply a database model or a table in the database. The columns in the tables would be the attributes we set in this model.
    
    ```go
    package models
    
    type Task struct {
    	ID          string `json:"id,omitempty"`
    	Title       string `json:"title"`
    	Description string `json:"description"`
    	Completed   bool   `json:"completed"`
    }
    ```
    
2. Now the most important implementation file. In the `handlers/task.go` file, paste the below code which handles all the interaction with the database. This is the file which would process all the incoming requests and serve the response accordingly.
    
    ```go
    package handlers
    
    import (
    	"fmt"
    	"net/http"
    
    	"github.com/Atoo35/gingonic-surrealdb/database"
    	"github.com/Atoo35/gingonic-surrealdb/models"
    	"github.com/gin-gonic/gin"
    	"github.com/surrealdb/surrealdb.go"
    )
    
    type TaskHandler struct {
    	DB *surrealdb.DB
    }
    
    func (h *TaskHandler) GetTasks(c *gin.Context) {
    	tasks, err := database.DB.Select("tasks")
    	if err != nil {
    		c.JSON(http.StatusInternalServerError, gin.H{
    			"message": fmt.Sprintf("Error while selecting tasks: %s", err.Error()),
    		})
    		return
    	}
    
    	tasksSlice := new([]models.Task)
    	err = surrealdb.Unmarshal(tasks, &tasksSlice)
    	if err != nil {
    		c.JSON(http.StatusInternalServerError, gin.H{
    			"message": fmt.Sprintf("Error while unmarshalling tasks: %s", err.Error()),
    		})
    		return
    	}
    
    	c.JSON(http.StatusAccepted, gin.H{
    		"tasks": tasksSlice,
    	})
    }
    
    func (h *TaskHandler) CreateTask(c *gin.Context) {
    	task := new(models.Task)
    	if err := c.ShouldBindJSON(task); err != nil {
    		c.JSON(http.StatusBadRequest, gin.H{
    			"message": fmt.Sprintf("Error while binding json: %s", err.Error()),
    		})
    		return
    	}
    
    	_, err := database.DB.Create("tasks", task)
    	if err != nil {
    		c.JSON(http.StatusInternalServerError, gin.H{
    			"message": fmt.Sprintf("Error while creating task: %s", err.Error()),
    		})
    		return
    	}
    	c.JSON(http.StatusCreated, gin.H{
    		"task": task,
    	})
    }
    
    func getSingletask(id string) (interface{}, error) {
    	task, err := database.DB.Select(id)
    	if err != nil {
    		return nil, err
    	}
    	fmt.Println(task)
    	return task, nil
    }
    
    func (h *TaskHandler) GetTask(c *gin.Context) {
    	id := c.Param("id")
    	task, err := getSingletask(id)
    	if err != nil {
    		c.JSON(http.StatusInternalServerError, gin.H{
    			"message": fmt.Sprintf("Error while getting task: %s", err.Error()),
    		})
    		return
    	}
    
    	taskModel := new(models.Task)
    	err = surrealdb.Unmarshal(task, &taskModel)
    	if err != nil {
    		c.JSON(http.StatusInternalServerError, gin.H{
    			"message": fmt.Sprintf("Error while unmarshalling task: %s", err.Error()),
    		})
    		return
    	}
    
    	c.JSON(http.StatusAccepted, gin.H{
    		"task": taskModel,
    	})
    }
    
    func (h *TaskHandler) UpdateTask(c *gin.Context) {
    	id := c.Param("id")
    	task := new(models.Task)
    	if err := c.ShouldBindJSON(task); err != nil {
    		c.JSON(http.StatusBadRequest, gin.H{
    			"message": fmt.Sprintf("Error while binding json: %s", err.Error()),
    		})
    		return
    	}
    
    	_, err := getSingletask(id)
    	if err != nil {
    		c.JSON(http.StatusInternalServerError, gin.H{
    			"message": fmt.Sprintf("Error while getting task: %s", err.Error()),
    		})
    		return
    	}
    
    	if err != nil {
    		c.JSON(http.StatusInternalServerError, gin.H{
    			"message": fmt.Sprintf("Error while unmarshalling task: %s", err.Error()),
    		})
    		return
    	}
    
    	_, err = database.DB.Update(id, task)
    	if err != nil {
    		c.JSON(http.StatusInternalServerError, gin.H{
    			"message": fmt.Sprintf("Error while updating task: %s", err.Error()),
    		})
    		return
    	}
    
    	c.JSON(http.StatusAccepted, gin.H{
    		"task": task,
    	})
    }
    
    func (h *TaskHandler) DeleteTask(c *gin.Context) {
    	id := c.Param("id")
    	_, err := database.DB.Delete(id)
    	if err != nil {
    		c.JSON(http.StatusInternalServerError, gin.H{
    			"message": fmt.Sprintf("Error while deleting task: %s", err.Error()),
    		})
    		return
    	}
    
    	c.JSON(http.StatusAccepted, gin.H{
    		"message": fmt.Sprintf("Task %s deleted", id),
    	})
    }
    ```
    
    This entire file has 5 main functions, `GetTasks`, `GetTask` (single task),`CreateTask`, `UpdateTask`, `DeleteTask`. All the functions are relatively simple and just interact with the database correspondingly and serve the result or error if any. If you need some help in understanding the code please reach out to me or comment on the post and I would be happy to help you.
    
    > Note: You would need to install gin/gonic here by running `go get install "github.com/gin-gonic/gin"`
    
3. The next file is `routes/routes.go`. In this file we simply redirect the incoming request to the corresponding handler. Here is the code.
    
    ```go
    package routes
    
    import (
    	"github.com/Atoo35/gingonic-surrealdb/api/handlers"
    	"github.com/gin-gonic/gin"
    	"github.com/surrealdb/surrealdb.go"
    )
    
    func SetupRoutes(db *surrealdb.DB) *gin.Engine {
    	h := &handlers.TaskHandler{DB: db}
    	router := gin.Default()
    
    	tasksRoutes := router.Group("/api/tasks")
    	{
    		tasksRoutes.GET("/", h.GetTasks)
    		tasksRoutes.POST("/", h.CreateTask)
    		tasksRoutes.GET("/:id", h.GetTask)
    		tasksRoutes.PUT("/:id", h.UpdateTask)
    		tasksRoutes.DELETE("/:id", h.DeleteTask)
    	}
    
    	return router
    }
    ```
    
    We are grouping all the `tasks` related APIs by using the `router.Group()` function provided by gin/gonic for creating readable and simple routes.
    
4. The final file is the `main.go` file in the root of the project. This file initializes the database and connects to it and starts the server.
    
    ```go
    package main
    
    import (
    	"log"
    
    	"github.com/Atoo35/gingonic-surrealdb/api/routes"
    	"github.com/Atoo35/gingonic-surrealdb/database"
    )
    
    func init() {
    	database.Connect("ws://localhost:8000/rpc", "root", "root")
    }
    
    func main() {
    	log.Printf("Woohooo")
    	router := routes.SetupRoutes(database.DB)
    
    	log.Println("Starting server on port 8080")
    	log.Fatal(router.Run(":8080"))
    	defer database.DB.Close()
    }
    ```
    
    To run the application simply run `go run main.go` and you are set. The server should be running on `http://localhost:8080` and you can now test the APIs by using postman or anything else if you prefer.
    

Do let me know in the comments if something is not working as expected or if there is some issue.

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