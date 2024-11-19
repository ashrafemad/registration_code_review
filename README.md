Reviewing this registration function:
![](/registration.png)

## Issues and resolution:

1- The password is stored as plain text (security issue)

    Solution: hash the password before storing it in the object
    
2- no return in case of another request method (GET for example)

    Solution: return response with 405 status code to tell the client this method is not allowed

3- no validation for inputs (if needed)

    Solution: Use serializer or form to validate the data before continuing with object creation

4- already existing users case is not handled

    Solution:
        - check for existence in the serializer/form while cleaning
        - handle IntegrityError for unique constraint then return a response with 400 status code and error message

5- improvement 1
  
    return created user as serialized object (eg. json object) instead of a message saying User created

6- improvement 2
    
    use restframework views instead of function based views for cleaner/organized code

## Revised solution:

```python3
class UserSerializer(serializers.ModelSerializer):
    password = serializers.CharField(
        write_only=True,
        required=True
    )

    class Meta:
        model = User
        fields = ("username", "email", "password")
    
    def validate_password(self, value):
        # Validate minimum/maximum length or any othe constraints
        return value

    def validate_username(self, value):
        if User.objects.filter(username=value).exists():
            raise ValidationError
        return value

    def create(self, validated_data):
        validated_data['password'] = make_password(validated_data['password'])
        return super().create(validated_data)


class UserRegistrationAPI(generics.CreateAPIView):
    serializer_class = UserSerializer
```
