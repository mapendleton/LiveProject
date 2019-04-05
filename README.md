# LiveProject
## Introduction
For the Tech Academy, I participated in a three week long Live Project. 
This project was our chance, as students of the Tech Academy, to
utilize all of the skills and knowledge that we had accumulated during
the program. The project was a full scale MVC Web Application in C#, 
including html and js. We worked as a team to tackle front end and back
end stories to add features to the project. I learned a great many things
working on this project. This was the largest project I've worked on, and
I had to make sure I had a solid understanding of the project in order to 
add features to it. That was a challenge, as some of the code was unfamiliar
to me, but because of that I was able to learn a great deal before adding
anything myself. I ran into some problems, but was able to look to my team
for assistance. It was challenging but rewarding work, and in the end I was 
able to create code that I am proud of. You can find examples of my work below.
## Stories
* [Schedule View](#schedule-view)
* [Create Schedule](#create-schedule)

### Schedule View
For this story I was tasked with pulling schedule data from a database to be displayed
on the schedule view so the user could see any schedules displayed on a calendar.
The project already had the necessary files to use a third party, open source, js,
event calendar called FullCalendar. After familiarizing myself with FullCalendar I
knew I had to create a schedule view model.
```
namespace Schedule_It_2.Models
{
    public class ScheduleViewModel
    {
        public DateTime ScheduleStartTime { get; set; }
        public DateTime? ScheduleEndTime { get; set; }
        public string UserName { get; set; }
        public List<ScheduledWorkPeriod> ScheduledWorkPeriods { get; set; }
    }
}
```
In the schedule controller I created a new list of schedule view model objects, 
pulled the necessary data from the data base and populated the list
then sent the data to the schedule view.
```
public class ScheduleController : Controller
    {
        private ApplicationDbContext db = new ApplicationDbContext();

        // GET: Schedule
        //public ActionResult Index()
        public ActionResult Index()
        {
            //Get data from schedules and users tables
            var schedules = db.Schedules.ToList();
            var users = db.Users.ToList();
            List<ScheduleViewModel> scheduleViews = new List<ScheduleViewModel>();

            //fills in schedule view model list to display on calendar
            //have not added ApprovedTimeOffEvents yet
            foreach (var schedule in schedules)
            {
                ScheduleViewModel scheduleView = new ScheduleViewModel();
                var user = db.Users.Where(x => x.Id == schedule.UserId).First();
                scheduleView.ScheduleStartTime = schedule.ScheduleStartDay;
                scheduleView.ScheduleEndTime = schedule.ScheduleEndDay;
                scheduleView.UserName = user.UserName;
                scheduleView.ScheduledWorkPeriods = db.ScheduledWorkPeriods.Where(x => x.ScheduleId == schedule.ScheduleId).ToList();

                scheduleViews.Add(scheduleView);
            }

            return View(scheduleViews);
        }
```
Taking the list of schedule view objects in the view, I converted it into JSON 
and stored it in an html hidden input tag so I could retrieve the data in a separate js file.
I then used the list of schedule views to create FullCalendar event objects to be displayed on
their calendar. 
```
//Schedule/Index calendar 
$(function () {
    //render calendar in view
    $('#scheduleCalendar').fullCalendar({
        displayEventEnd: true,
        eventLimit: true
    })
    //JSON request to pull data from Schedule table, write to list and send to calendar.
    let viewList = []
    let queryList = JSON.parse($('#scheduleData').val()); 

    for (let x of queryList) {
       
        viewList.push({
            title: x.UserName,
            start: moment(x.ScheduleStartTime),
            end: moment(x.ScheduleEndTime)
        });
        }         
    $('#scheduleCalendar').fullCalendar('renderEvents', viewList, true);
})
```
### Create Schedule
This story had me work on both the front end and back end, however, for front end part
I used a barebones design mainly so I could focus on the back end functionality. I created
an html post form with 7 sections for the 7 days of the week that stored user input into an
array. When the user finishes the weekday form, a modal will pop up asking for for weekly info. 
```
<div class="row">
    <div class="col">
        @using (Html.BeginForm("Create", "Schedule", FormMethod.Post))
        {
        <form>
            @* info for each day is bound to an element in an array that gets sent back to the controller *@
            <div>
                <h1>Monday</h1>
                <div class="form-group">
                    <label for="startTime">Start Time</label>
                    <input name="scheduledWorkPeriods[0].StartTime" type="time" class="form-control" id="startTime" aria-describedby="timeHelp">
                    <small id="timeHelp" class="form-text text-muted">Enter the time you start for this day.</small>
                </div>
                <div class="form-group">
                    <label for="endTime">End Time</label>
                    <input name="scheduledWorkPeriods[0].EndTime" type="time" class="form-control" id="endTime">
                </div>
                @*<div class="form-group form-check">
                <input name="scheduledWorkPeriods[0].IsDayOff" type="checkbox" class="form-check-input" id="day-off">
                <label class="form-check-label" for="day-off">Day off?</label>
            </div>*@

                @* Had to use multiple select instead of checkbox because checkbox wasn't working *@
                <div class="form-group">
                    <label for="isDayOff">Example select</label>
                    <select name="scheduledWorkPeriods[0].IsDayOff" class="form-control" id="isDayOff">
                        <option>false</option>
                        <option>true</option>
                    </select>
                </div>
                <input name="scheduledWorkPeriods[0].Day" type="hidden" value="Mon" />
            </div>
            <div>
                <h1>Tuesday</h1>
                <div class="form-group">
                    <label for="startTime">Start Time</label>
                    <input name="scheduledWorkPeriods[1].StartTime" type="time" class="form-control" id="startTime" aria-describedby="timeHelp">
                    <small id="timeHelp" class="form-text text-muted">Enter the time you start for this day.</small>
                </div>
                <div class="form-group">
                    <label for="endTime">End Time</label>
                    <input name="scheduledWorkPeriods[1].EndTime" type="time" class="form-control" id="endTime">
                </div>
                <div class="form-group">
                    <label for="isDayOff">Example select</label>
                    <select name="scheduledWorkPeriods[1].IsDayOff" class="form-control" id="isDayOff">
                        <option>false</option>
                        <option>true</option>
                    </select>
                </div>
                <input name="scheduledWorkPeriods[1].Day" type="hidden" value="Tue" />

            </div>
            <div>
                <h1>Wednesday</h1>
                <div class="form-group">
                    <label for="startTime">Start Time</label>
                    <input name="scheduledWorkPeriods[2].StartTime" type="time" class="form-control" id="startTime" aria-describedby="timeHelp">
                    <small id="timeHelp" class="form-text text-muted">Enter the time you start for this day.</small>
                </div>
                <div class="form-group">
                    <label for="endTime">End Time</label>
                    <input name="scheduledWorkPeriods[2].EndTime" type="time" class="form-control" id="endTime">
                </div>
                <div class="form-group">
                    <label for="isDayOff">Example select</label>
                    <select name="scheduledWorkPeriods[2].IsDayOff" class="form-control" id="isDayOff">
                        <option>false</option>
                        <option>true</option>
                    </select>
                </div>
                <input name="scheduledWorkPeriods[2].Day" type="hidden" value="Wed" />

            </div>
            <div>
                <h1>Thursday</h1>
                <div class="form-group">
                    <label for="startTime">Start Time</label>
                    <input name="scheduledWorkPeriods[3].StartTime" type="time" class="form-control" id="startTime" aria-describedby="timeHelp">
                    <small id="timeHelp" class="form-text text-muted">Enter the time you start for this day.</small>
                </div>
                <div class="form-group">
                    <label for="endTime">End Time</label>
                    <input name="scheduledWorkPeriods[3].EndTime" type="time" class="form-control" id="endTime">
                </div>
                <div class="form-group">
                    <label for="isDayOff">Example select</label>
                    <select name="scheduledWorkPeriods[3].IsDayOff" class="form-control" id="isDayOff">
                        <option>false</option>
                        <option>true</option>
                    </select>
                </div>
                <input name="scheduledWorkPeriods[3].Day" type="hidden" value="Thur" />

            </div>
            <div>
                <h1>Friday</h1>
                <div class="form-group">
                    <label for="startTime">Start Time</label>
                    <input name="scheduledWorkPeriods[4].StartTime" type="time" class="form-control" id="startTime" aria-describedby="timeHelp">
                    <small id="timeHelp" class="form-text text-muted">Enter the time you start for this day.</small>
                </div>
                <div class="form-group">
                    <label for="endTime">End Time</label>
                    <input name="scheduledWorkPeriods[4].EndTime" type="time" class="form-control" id="endTime">
                </div>
                <div class="form-group">
                    <label for="isDayOff">Example select</label>
                    <select name="scheduledWorkPeriods[4].IsDayOff" class="form-control" id="isDayOff">
                        <option>false</option>
                        <option>true</option>
                    </select>
                </div>
                <input name="scheduledWorkPeriods[4].Day" type="hidden" value="Fri" />

            </div>
            <div>
                <h1>Saturday</h1>
                <div class="form-group">
                    <label for="startTime">Start Time</label>
                    <input name="scheduledWorkPeriods[5].StartTime" type="time" class="form-control" id="startTime" aria-describedby="timeHelp">
                    <small id="timeHelp" class="form-text text-muted">Enter the time you start for this day.</small>
                </div>
                <div class="form-group">
                    <label for="endTime">End Time</label>
                    <input name="scheduledWorkPeriods[5].EndTime" type="time" class="form-control" id="endTime">
                </div>
                <div class="form-group">
                    <label for="isDayOff">Example select</label>
                    <select name="scheduledWorkPeriods[5].IsDayOff" class="form-control" id="isDayOff">
                        <option>false</option>
                        <option>true</option>
                    </select>
                </div>
                <input name="scheduledWorkPeriods[5].Day" type="hidden" value="Sat" />

            </div>
            <div>
                <h1>Sunday</h1>
                <div class="form-group">
                    <label for="startTime">Start Time</label>
                    <input name="scheduledWorkPeriods[6].StartTime" type="time" class="form-control" id="startTime" aria-describedby="timeHelp">
                    <small id="timeHelp" class="form-text text-muted">Enter the time you start for this day.</small>
                </div>
                <div class="form-group">
                    <label for="endTime">End Time</label>
                    <input name="scheduledWorkPeriods[6].EndTime" type="time" class="form-control" id="endTime">
                </div>
                <div class="form-group">
                    <label for="isDayOff">Example select</label>
                    <select name="scheduledWorkPeriods[6].IsDayOff" class="form-control" id="isDayOff">
                        <option>false</option>
                        <option>true</option>
                    </select>
                </div>
                <input name="scheduledWorkPeriods[6].Day" type="hidden" value="Sun" />

            </div>
            @* this button brings up the modal *@
            <button type="button" class="btn btn-primary" data-toggle="modal" data-target="#ScheduleModal">Submit</button>

            <div class="modal" id="ScheduleModal" tabindex="-1" role="dialog">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">Modal title</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <div>
                            <div class="form-group">
                                <label for="startDate">Start Date</label>
                                <input name="startDate" type="datetime" class="form-control" id="startDate">
                            </div>
                            <div class="form-group">
                                <label for="endDate">End Date (Optional)</label>
                                <input name="endDate" type="datetime" class="form-control" id="endDate">
                            </div>
                            <div class="form-group form-check">
                                <input name="notes" type="text" class="form-check-input" id="notes">
                                <label class="form-check-label" for="notes">Notes</label>
                            </div>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        <button type="submit" class="btn btn-primary">Submit</button>
                    </div>
                </div>
            </div>
        </div>
        </form>
        }

    </div>
</div>
```
In the Schedule Controller, the HttpPost method "Create" takes the user input array as an array 
of ScheduleWorkPeriod models. Using that information the method updates the schedule database and
sends the user back to the index where they can see the new schedule on the calendar in the schedule view.
```
[HttpPost]
public ActionResult Create(ScheduledWorkPeriod[] scheduledWorkPeriods, DateTime startDate, DateTime endDate, string notes)
{
    //if a schedule is created directly from the create view without a user id it breaks the index view 
    if (ModelState.IsValid)
    {
        Schedule schedule = new Schedule() 
        { 
          ScheduleStartDay = startDate, 
          ScheduleEndDay = endDate, 
          Notes = notes, 
          ScheduledWorkPeriods = new List<ScheduledWorkPeriod>()
        };
        schedule.ScheduleId = Guid.NewGuid();
        schedule.UserId = HttpContext.User.Identity.GetUserId();


        foreach (var sched in scheduledWorkPeriods)
        {
            ScheduledWorkPeriod scheduledWorkPeriod = new ScheduledWorkPeriod()
            {
                StartTime = sched.StartTime,
                EndTime = sched.EndTime,
                ScheduleId = schedule.ScheduleId,
                Day = sched.Day,
                IsDayOff = sched.IsDayOff
            };
            schedule.ScheduledWorkPeriods.Add(scheduledWorkPeriod);
            db.ScheduledWorkPeriods.Add(scheduledWorkPeriod);
            db.Schedules.Add(schedule);
            db.SaveChanges();
            return RedirectToAction("Index");
        }
    }
    return View();
}
```
### Other Skills Learned
