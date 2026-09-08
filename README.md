# Task-Scheduler-Application
import json
import time
import threading
from datetime import datetime

TASK_FILE = "tasks.json"


# Load tasks from JSON file
def load_tasks():
    try:
        with open(TASK_FILE, "r") as file:
            return json.load(file)
    except (FileNotFoundError, json.JSONDecodeError):
        return []


# Save tasks to JSON file
def save_tasks(tasks):
    with open(TASK_FILE, "w") as file:
        json.dump(tasks, file, indent=4)


# Add a new task
def add_task():
    title = input("Enter task name: ")
    date = input("Enter date (YYYY-MM-DD): ")
    task_time = input("Enter time (HH:MM): ")

    try:
        datetime.strptime(
            f"{date} {task_time}",
            "%Y-%m-%d %H:%M"
        )
    except ValueError:
        print("Invalid date or time format.")
        return

    tasks = load_tasks()

    task = {
        "id": len(tasks) + 1,
        "title": title,
        "datetime": f"{date} {task_time}",
        "completed": False,
        "notified": False
    }

    tasks.append(task)
    save_tasks(tasks)

    print("Task added successfully!")


# Display all tasks
def view_tasks():
    tasks = load_tasks()

    if not tasks:
        print("\nNo tasks available.")
        return

    print("\n--------- TASKS ---------")

    for task in tasks:
        status = "Completed" if task["completed"] else "Pending"

        print(
            f"ID: {task['id']} | "
            f"Task: {task['title']} | "
            f"Time: {task['datetime']} | "
            f"Status: {status}"
        )


# Mark task as completed
def complete_task():
    tasks = load_tasks()

    if not tasks:
        print("No tasks available.")
        return

    view_tasks()

    try:
        task_id = int(input("\nEnter task ID to complete: "))
    except ValueError:
        print("Please enter a valid ID.")
        return

    for task in tasks:
        if task["id"] == task_id:
            task["completed"] = True
            save_tasks(tasks)

            print("Task marked as completed!")
            return

    print("Task not found.")


# Delete task
def delete_task():
    tasks = load_tasks()

    if not tasks:
        print("No tasks available.")
        return

    view_tasks()

    try:
        task_id = int(input("\nEnter task ID to delete: "))
    except ValueError:
        print("Please enter a valid ID.")
        return

    new_tasks = [
        task for task in tasks
        if task["id"] != task_id
    ]

    if len(new_tasks) == len(tasks):
        print("Task not found.")
        return

    # Reassign IDs
    for index, task in enumerate(new_tasks, start=1):
        task["id"] = index

    save_tasks(new_tasks)

    print("Task deleted successfully!")


# Background reminder system
def reminder_system():
    while True:
        tasks = load_tasks()
        current_time = datetime.now().strftime("%Y-%m-%d %H:%M")

        updated = False

        for task in tasks:

            if (
                task["datetime"] == current_time
                and not task["completed"]
                and not task["notified"]
            ):
                print(
                    f"\n🔔 REMINDER: "
                    f"{task['title']} is due now!"
                )

                task["notified"] = True
                updated = True

        if updated:
            save_tasks(tasks)

        time.sleep(30)


# Main menu
def main():
    reminder_thread = threading.Thread(
        target=reminder_system,
        daemon=True
    )

    reminder_thread.start()

    while True:

        print("\n==========================")
        print("      TASK SCHEDULER")
        print("==========================")
        print("1. Add Task")
        print("2. View Tasks")
        print("3. Complete Task")
        print("4. Delete Task")
        print("5. Exit")

        choice = input("\nEnter your choice: ")

        if choice == "1":
            add_task()

        elif choice == "2":
            view_tasks()

        elif choice == "3":
            complete_task()

        elif choice == "4":
            delete_task()

        elif choice == "5":
            print("Thank you for using Task Scheduler!")
            break

        else:
            print("Invalid choice. Please try again.")


if __name__ == "__main__":
    main()
