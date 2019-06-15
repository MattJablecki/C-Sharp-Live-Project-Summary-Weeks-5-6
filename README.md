# C-Sharp-Live-Project-Summary-Weeks-5-6

## Introduction
For my third two-week sprint on the Erectors Construction Project, I was assigned specifically to Back-End Development. In my previous sprint, I had been tasked with creating the Front-End for a drag and drop schedule creator. For a look at this assignment, click this link: https://github.com/MattJablecki/C-Sharp-Live-Project-Summary-Weeks-3-4.

This sprint was dedicated to implementing the Back-End functionality to make the drag and drop schedule creator operational. 

Below you'll find a brief description of how this was achieved, including the code I used to make it happen.


### Back-End Drag and Drop Functionality
The first challenge I encountered on this sprint was capturing the user's end result after setting the date range and assigning employees to their respective jobs. This was achieved using JavaScript to identify where the pertinent data existed and then formatting it in a way that allowed me to manipuate it as necessary. First, every schedule has a start date and an end date. Utilizing the drop down created in the previous sprint, I then targeted the user's selected value like so:

            var week = $('#weekDDL :selected').text();

Next, I needed to collect all instances of job and place them in an array:


            $("#jobBox h3").each(function () {
                    var $this = $(this);
                    valuearray.push($this.attr("id"));
                    return valuearray;
                });


Within each job box, there was a list of employees that received the drag and drop employee items. Collecting those was achieved in the same way as collecting the jobs:

            $("#jobBox li").each(function () {
                var $this = $(this);
                idarray.push($this.attr("id"));
            });

With two arrays, one for job and one for employees, the new challenge became accurately combining the two in such a way as to delineate which employees were assigned to which job. Each list of employees under instance of job started with a dummy list item that instructed users where the drop target was. Leveraging that list item, an object was created in which the value for all jobs and employees were inserted into an array, but any instance of the dummy list item was removed, resulting in a well organized array in which each instance of job was followed directly by the employees that were inserted into the associated list. The resulting code looks like this:


            function buildObj(namesArr, jobsArr) {
                jobsArr.shift();
                for (var i = 0; i < namesArr.length; i++) {
                    if (namesArr[i] === 'no') {
                        namesArr[i] = jobsArr[0];
                        jobsArr.shift();
                    }
                }
                //creates new array
                return namesArr;
            }
       
An unforeseen bug was that the dummy list item was still sortable, meaning that if an employee was inserted into the list above that item, it threw the organization of the array out of whack. I was able to modify the .sortable like so:

        $(document).ready(function () {
            $(".sortable_list").sortable({ connectWith: ".connectedSortable", items: ("> li", "li:not(#no)") });
        });

This locked the dummy list item in position, disabling the sortable property and allowing for correct array assembly. Now that the targeted data was collected, it was time to pass to the controller. This was achieved with a simple AJAX call. But in order for the data to be sent properly, it needed to be converted to JSON, like so:

       var myJsonString = JSON.stringify(buildObj(idarray, valuearray));


Once the data was converted to JSON data, I then sent the data to the controller using an AJAX call triggered by clicking the submit button:

           $.ajax({
                type: "POST",
                url: "/Schedules/GetData",
                datatype: "JSON",
                data: {
                    namesArr: myJsonString,
                    weeks: week
                },
            });
            
The JSON string, when passed to the controller, then needed to be parsed in order to convert it back to an array. In order to do so, I first needed to identify and remove any unnecessary characters from the string:

            char[] myChar = { '[', ']' };
            string newEmployee = employees.Trim(myChar);
            
With the characters removed, I then identified the point at which each item in the string starts/ends and assigned it back to an array:

            string[] employeeList = newEmployee.Split(',');
            var listEmployees = employeeList.ToList<string>();
            
Similarly, I had to separate the JSON string containing the date range and parse it to assign values for a schedule's start date and end date:

            string[] stringSeperator = new string[] {"to"};          
            string[] dateRange = weeks.Split(stringSeperator, StringSplitOptions.None);
            string startDate = dateRange[0];
            string endDate = dateRange[1];            
