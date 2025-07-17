# Models
## Helper Fnctions: 
### get_user_role(self)-> returns self.userprofile.role
    User.role: extra property added to django.contrib.auth.models' User model. 
    some_user.role invokes get_user_role.
### user_directory_path
    returns full path for profile_picture
### no_whitespace
    a helper function thta checks whether the string contains only tabs and whitespace characters. 

## Model
### UserExtension: A model with two fields 
    user: OneToOneField to User
    created_by: Foreign key to User model
    created automatically everytime a User object is created.
### CleanValidateModel:
    an abstract model that is base model for all other models 
    it intelligenly skips full_clean() if the data is passed through serializer and is validated. 
    a class level flag field _validated, default false, serialziers sets it as True if the data is validated.
    avoids redundant same validation process at database level while saveing model. 
    created: models.DateTime autofield and assigned when the object is created

### UserProfile extends CleanValidatedModel

    This model has following fields:
    1. status: Whether the profile is active, suspended or deleted
        ACT, SUS, DEL
    2. role: Indicates the role of the profile (Admin, staff and Creator)
        AD, ST, CR
    3. display_name(max 65 char): This name will be used as display name in UI. Default=full name
        and must contain only alphabets. 
    4. first_name: First Name (32 char alphabets only) only alphabet characters
    5. last_name: Last Name (32 char alphabets only) only alphabet characters
    6. email: must be in valid e-mail format and unique for each
    7. phone_number: 10 digit nepali phone number and unique
    8. user: one to one field to django.contrib.auth.models.User instance
    9. date_of_birth: Django Date field and age must be >= 18.
    10. profile_picture: Display picture, valid image file only 400x400. max size 5MB
    11. 
    12. 

# Serializers
## Helper Function
### check_role_assignment
    arguments: assigner_role, assigned_role
    returns True or False based on role Hierarchy: Admin>Staff>Creator
    True: assigner_role is Admin and assigned role is not Admin
    True: assigner_role is Staff and assigned roel is Creator
    False: otherwise


## Serialzier
### BaseModelSerialzier: 
    extends serialziers.ModelSerialzier
    base model for other models
    provide create and update methods that sets Models._is_validated flag 
    also provides a validate method that validates if the user creating or
    updating the instance has permission to assign a role. 
    while creating/updating Admin can only set 'ST' or 'CR' Role, and Staff can set only 'CR'

### PublicProfileSerializer
    Model: UserProfile
    for listing user profiles with some user details.
    serializes ['first_name','last_name', 'display_name', 'email','public_id'] and are read_only. 

### UserProfileDetailSerializer
    for detail view of oneself's profile and one can update self profile 
    serializes:  'first_name','last_name', 'display_name', 'email','public_id',
            'phone_number','date_of_birth', 'username', 'profile_picture','role','status',
    and read_only_fields are 'public_id','phone_number', 'username','role','status' which are not editable to self. 

### SuperUpdateSerialzier:
    for updating userprofiles of the users lower in rank. Admin>Staff>Creator
    serializes: 'first_name','last_name', 'display_name', 'email','public_id',
            'phone_number','date_of_birth', 'username', 'profile_picture','role','status',
    read_only fields: 'public_id', 'username'

### CreateProfileSerializers
    To create a User Profile, Only Stadd User and Admin User can create UserProfile. 
    Admin can assign: CR or ST
    Staff can only assign CR. 
    role: A choice Field that serializes CR and ST for Admin User and Only CR for Staff User,
    status[Future Use]: Active, Suspended, Deleted only Active User can log into system, Deleted suspended User cannot log into system but His Contents are still visible to everyone. The Deleted Users Accounts Data and Profile are archieved and are unavailable for public

### CreateUserSerializer
    To Create a new user for django.contrib.auth.models.Users
    felds: password1, password2, username
    username: one cannot include `admin` in username
    must be at least 3 character long and max 12
    must start with alphabet followed by alphabet, digits or underscore only
    creates UserExtension object while creating a user

### Reset Password Serializer
    For resetting passwords of users other lower in rank
    Admin>Staff>Creator
    if not tied to any profile Admin or Staff can RESET PASSWORD no matter who created the User Object. 
### UpdatePasswordSerialzier:
    Only the self can Change the password, 
    new password must not match to current password. 



