# Exercises solutions
## Database validation with Great Expectations
### Exercise 1
```
home_data.expect_column_values_to_be_of_type("Street", 'str')
home_data.expect_column_values_to_not_be_null("LandContour")
home_data.expect_column_min_to_be_between("YearBuilt", 1700, 1900)
home_data.expect_column_median_to_be_between("LotArea", 5000, 15000)
home_data.expect_column_most_common_value_to_be_in_set("SaleType", ["WD", "New"])
```

## Training pipeline data validation with Pandera
### Exercise 1
```
exercise_schema = pa.DataFrameSchema(
    columns={
        "Id": pa.Column(int, required=True, nullable=False, unique=True),
        "MSZoning": pa.Column(str, checks=pa.Check.isin(['RL', 'RM', 'C (all)', 'RH', 'FV']), required=False, nullable=True),
        "OverallQual": pa.Column(int, required=True, nullable=False, checks=pa.Check.in_range(1, 10)),
        "BsmtCond": pa.Column(str, nullable=True, required=False, checks=pa.Check.str_length(2)),
        "1stFlrSF": pa.Column(int, nullable=True),
        "2ndFlrSF": pa.Column(int, nullable=True),
    },
    strict="filter",
    checks=pa.Check(lambda df: df["2ndFlrSF"].mean() <= df["1stFlrSF"].mean()),
)
```

## Model serving data validation with Pydantic
### Exercise 1
```
class FireplaceQu(str, Enum):
    Ex = 'Ex'
    Gd = 'Gd'
    TA = 'TA'
    Fa = 'Fa'
    Po = 'Po'

class Input(BaseModel):
    YearBuilt: int
    Fireplaces: Optional[int]
    FireplaceQu: Optional[FireplaceQu]

    @validator("FireplaceQu")
    @classmethod
    def validate_fire_places_quality_field(cls, field_value, values):
        if values["Fireplaces"]>0 and field_value is None:
            raise ValueError(f"FireplaceQu is required when Fireplaces is greater than 0")
        return field_value

    class Config:
        extra = 'forbid'
        allow_mutation = False
```