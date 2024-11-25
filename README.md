# Linux-Automation
Script that automates user managment

Paste this script into your linux terminal:

#!/bin/bash

# Exit script if any command fails
set -e

# Function to create a user
create_user() {
    read -p "Enter username: " username
    read -p "Enter home directory (leave blank for default): " home_dir
    read -p "Enter shell (leave blank for default): " user_shell
    read -p "Assign to groups (comma-separated, leave blank for none): " groups

    # Construct useradd command
    cmd="useradd $username"
    [[ -n "$home_dir" ]] && cmd="$cmd -d $home_dir"
    [[ -n "$user_shell" ]] && cmd="$cmd -s $user_shell"
    [[ -n "$groups" ]] && cmd="$cmd -G $groups"

    # Execute command
    eval "$cmd"
    echo "User $username created successfully."
}

# Function to delete a user
delete_user() {
    read -p "Enter username to delete: " username
    read -p "Delete home directory as well? (y/n): " delete_home

    if [[ "$delete_home" == "y" ]]; then
        userdel -r "$username"
    else
        userdel "$username"
    fi
    echo "User $username deleted successfully."
}

# Function to modify a user
modify_user() {
    read -p "Enter username to modify: " username
    echo "What would you like to modify?"
    echo "1. Change home directory"
    echo "2. Change shell"
    echo "3. Add to groups"
    echo "4. Remove from groups"
    read -p "Choose an option [1-4]: " option

    case $option in
        1)
            read -p "Enter new home directory: " new_home
            usermod -d "$new_home" "$username"
            echo "Home directory for $username changed to $new_home."
            ;;
        2)
            read -p "Enter new shell: " new_shell
            usermod -s "$new_shell" "$username"
            echo "Shell for $username changed to $new_shell."
            ;;
        3)
            read -p "Enter groups to add (comma-separated): " add_groups
            usermod -a -G "$add_groups" "$username"
            echo "User $username added to groups: $add_groups."
            ;;
        4)
            read -p "Enter groups to remove (comma-separated): " remove_groups
            for group in $(echo "$remove_groups" | tr ',' ' '); do
                gpasswd -d "$username" "$group"
            done
            echo "User $username removed from groups: $remove_groups."
            ;;
        *)
            echo "Invalid option."
            ;;
    esac
}

# Function to display user info
display_user_info() {
    read -p "Enter username to view information: " username
    if id "$username" >/dev/null 2>&1; then
        id "$username"
        grep "^$username:" /etc/passwd
    else
        echo "User $username does not exist."
    fi
}

# Main menu
while true; do
    echo "User Management Script"
    echo "-----------------------"
    echo "1. Create a new user"
    echo "2. Delete a user"
    echo "3. Modify a user"
    echo "4. Display user information"
    echo "5. Exit"
    read -p "Choose an option [1-5]: " choice

    case $choice in
        1)
            create_user
            ;;
        2)
            delete_user
            ;;
        3)
            modify_user
            ;;
        4)
            display_user_info
            ;;
        5)
            echo "Exiting script."
            exit 0
            ;;
        *)
            echo "Invalid option. Please try again."
            ;;
    esac
    echo
done
