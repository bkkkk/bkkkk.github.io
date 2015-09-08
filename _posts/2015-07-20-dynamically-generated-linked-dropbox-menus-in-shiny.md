---
layout: post
date: 2015-07-20
title: Dynamically generated linked dropbox menus with R and Shiny
author: Jacobo Blanco
tags: [rstats,shiny,r,webapp,webdev]
published: True
comments: True
---

I've been recently tasked with coming up with a time tracking solution for our team. We work on multiple projects, some of which are publicly funded, and this means we need to be able to track individual project contributions. Due to various technicalities it makes more sense to track percentage contribution instead of actual hours. In addition some projects have sub-projects for which we need to track contributions as well.

After wrestling with redmine with various plugins I decided that "there has to be a better way". I've been using R and Shiny for another project so I thought I would use my newly acquired R chops to build a Shiny (pun intended) time tracking web-application. This gives us all the power of R for any further manipulation we are likely to need in the future, but the simplicity of a nicely designed web interface.

The stars seemed to align and the excellent Dean Attali had recently released an article on using R and Shiny to build Google Form-like sites for data collection using R. Perfect! This became the basis for the web app, except I switched from CSV-based storage to MySQL, and I am pulling a bunch of the category data (project name, and user name) from the database.

All the code shown here is part of a minimal working example which I will upload later. I've noted where I normally query the mySQL database so you can go ahead a mirror that in your own code. Also note that this is my solution, not the best solution or the most concise.

## Dynamically generated UI

Each user needs to be able to submit their weekly project contribution against any number of projects. There are multiple design solutions to this problem, like having add/remove buttons to add a contribution to a project. This approach has too many sources of error so I decided to simply ask the user how many project they would like to record and then generate N number of contribution fields.

Each contribution requires the user answer three questions: Which project is it? Which sub-project, if any, is it? What is the size of the contribution? So there are three fields, one for each question. A group of fields is generated for the number of *N* contributions a user would like to track.

The three elements are dynamically generated in `server.R`. The first snippet of code generates a list of input objects for each contribution to be recorded:

{% highlight r %}
build_fields <- function(entries_to_add) {
  list_of_projects <- # LOAD DATA FROM mySQL

  lapply(1:entries_to_add, function(entry) {
    list(
      column(4, selectInput(paste0("project_field", entry), "Project Name:", choices = c("", list_of_projects))),
      column(4, selectInput(paste0("subproject_field", entry), "Sub-project Name:", choices = c(""))),
      column(4, sliderInput(paste0("contrib_field", entry), "How much of week contributed:", min = 0, max = 100, step = 5, value = 5, post = "%"))
    )
  })
}

contribution_ui_generator <- eventReactive(input$add_more, {
  entries <- input$number_of_contributions
  build_fields(entries)
})

output$ui_contributions_fields <- renderUI({
  contribution_ui_generator()
})
{% endhighlight %}

And then displayed in the `ui.R`:

{% highlight r %}
user_form <- 
  div(id = "form",
      fluidRow(
        column(4, selectInput("user_name", "Select Name of Reseacher:", choices = users)),
        column(4, sliderInput("number_of_projects", "How many contributions to add?", min = 1, max = 10, value = 2))
      ),
      fluidRow(
        column(4, actionButton("add_more", "Add Contributions"))
      ),
      hr()
)

shinyUI(fluidPage(
  titlePanel("Example Dynamic UI"),
  user_form,
  uiOutput("ui_contributions_fields")
))
{% endhighlight %}

## Updating menu content

Next the sub-projects field needs to update depending on which project is selected. You've probably seen these kinds of dependent or linked dropbox menus, especially with country and state/county fields in registration forms. When you select USA, the state menu is populated with the American states, if you select UK you get a list of counties, and so on.

All the sub-project information is stored on the MySQL database so I pull that information using the `RMySQL` package, apply a simple filter by projectID and select only the name column. This gives me the list of sub-projects associated with the selected project.

The content of a selectInput can be updated using the `updateSelectInput()` function as follows:

{% highlight r %}
updateSelectInput("subproject_field", choices = c("new", "choices"))
{% endhighlight %}

## Creating an observer

In order to react to changes in the project name field I create an observer, that updates the choices in the sub-projects field when the value of the project field changes:

{% highlight r %}
observe({
  project_id <- input$project_field
  subprojects <- # LOAD DATA FROM mySQL

  list_of_subprojects <-
    subprojects %>%
      filter(projectID == selected_project) %>%
      .[['name']]

  updateSelectInput("subproject_field", choices = list_of_subprojects)
})
{% endhighlight %}

But then I run into a problem: I can build dynamic UI elements, I can create observers for known input objects, how does one create observers for an arbitrary number of objects?

## Bringing it all together

The trick here is to realize that you can create observer objects in a loop and also nest reactive and observer objects.

So after all the fields are built we generate an observer connecting each project dropbox to a subproject dropbox. There are a lot of parts to the following code so first we loop over all the generated entries and create an *observeEvent* object that reacts to changes in the project dropbox selection, loading the subproject data, filters out rows where the project name matches the project name selected in the dropbox menu. Then the subproject dropbox menu choices are updated to include all the associated subprojects.

You'll note that I am using the local object here, this is to bind the value of entry to the current iteration, otherwise only one of the dropbox menus is updated. Finally I return a `div` object containing all the contribution fields:

{% highlight r %}
contribution_ui_generator <- eventReactive(input$add_more, {
  entries_to_add <- input$number_of_projects
  contribution_fields <- build_fields(input$number_of_projects)
  
  for(entry in 1:entries_to_add) {
    local({
      local_entry <- entry
      project_name_field <- paste0("project_field", local_entry)
      
      observeEvent(input[[project_name_field]], {
        local_project_field_name <- project_name_field
        subproject_field_name <- paste0("subproject_field", local_entry)

        selected_project <- input[[local_project_field_name]]
        
        list_of_subprojects <- subprojects %>%
          filter(projectID == selected_project) %>%
          .[['name']]
        
        updateSelectInput(session, subproject_field_name, choices = list_of_subprojects)
      })

    })
  }

  div(id = "list_of_contributions", contribution_fields)
})
{% endhighlight %}

This should set you up with dynamically generated, linked dropbox menus. This technique can be used to create observers for most types of dynamically generated UI elements.

Thank you for reading, I hope this was helpful. Let me know if you've found a better way or have any questions.

