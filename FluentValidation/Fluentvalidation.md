# FluentValidation 

### Example 
```c#
public class CustomerValidator : AbstractValidator<Customer>
{
  public CustomerValidator()
  {
    RuleFor(x => x.Surname).NotEmpty();
    RuleFor(x => x.Forename).NotEmpty().WithMessage("Please specify a first name");
    RuleFor(x => x.Discount).NotEqual(0).When(x => x.HasDiscount);
    RuleFor(x => x.Address).Length(20, 250);
    RuleFor(x => x.Postcode).Must(BeAValidPostcode).WithMessage("Please specify a valid postcode");
  }

  private bool BeAValidPostcode(string postcode)
  {
    // custom postcode validating logic goes here
  }
}

public class Customer 
{
  public int Id { get; set; }
  public string Surname { get; set; }
  public string Forename { get; set; }
  public decimal Discount { get; set; }
  public string Address { get; set; }
}

```


[Creating your first validator — FluentValidation documentation](https://docs.fluentvalidation.net/en/latest/start.html#)

To run the validator, instantiate the validator object and call the `validate` method, passing in the object to validate. 

```c#
Customer customer = new Customer();
CustomerValidator validator = new CustomerValidator(); 

ValidationResult result = validator.Validate(customer); 
```

The `Validate` method returns a ValidationResult object. This contains two properties: 

- `IsValid` - a boolean that says whether the validation succeeded. 
- Errors - a collection of ValidationFailure objects containing details about any validation failures. 

You can also call `ToString` on the `ValidationResult` to combine all error messages into a single sstring. By default, the messages will be separated with new lines, but if you want to customize the behaviour you can pass a different separator character to `ToString`. 

```C#
ValidationResult results = validator.Validate(customer); 
string allMessages = results.ToSTring('~'); 
o
```