
_# An admin email needs to be set for Moodle_  
_while true; do read -p "Enter email: " USER_EMAIL; "$USER_EMAIL" =~ ^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,4} && break || echo "Invalid email address. Try again."; done_