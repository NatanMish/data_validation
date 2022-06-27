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

### Exercise 2
```
from typing import Dict, Optional

from great_expectations.core.expectation_configuration import ExpectationConfiguration
from great_expectations.exceptions.exceptions import (
    InvalidExpectationConfigurationError,
)
from great_expectations.execution_engine import (
    ExecutionEngine,
    PandasExecutionEngine,
    SparkDFExecutionEngine,
    SqlAlchemyExecutionEngine,
)
from great_expectations.expectations.expectation import (
    ColumnPairMapExpectation,
    ExpectationValidationResult,
)
from great_expectations.expectations.metrics.import_manager import F, sa
from great_expectations.expectations.metrics.map_metric_provider import (
    ColumnPairMapMetricProvider,
    column_pair_condition_partial,
)
from great_expectations.validator.metric_configuration import MetricConfiguration


class ColumnFloorsSquareFeetComparison(ColumnPairMapMetricProvider):
    """MetricProvider Class for columns floors square feet comparison"""
    condition_metric_name = "column_pair_values.floors_square_feet_ratio"
    condition_domain_keys = (
        "column_A",
        "column_B",
    )
    condition_value_keys = ()
    @column_pair_condition_partial(engine=PandasExecutionEngine)
    def _pandas(cls, column_A, column_B, **kwargs):
        # This methold should return a Pandas series of booleans
        return column_B/column_A <= 2


class ExpectProportionalFloorDifference(ColumnPairMapExpectation):
    """Expect house 2nd floor to be no more than twice larger than the 1st floor"""
    map_metric = "column_pair_values.floors_square_feet_ratio"
    # These examples will be shown in the public gallery.
    # They will also be executed as unit tests for your Expectation.
    examples = [
        {
            "data": {
                "col_a": [1000, 500, 2000, 4000, 300, 100],
                "col_b": [500, 1000, 1000, 2500, 0, 2000],
            },
            "tests": [
                {
                    "title": "basic_positive_test",
                    "exact_match_out": False,
                    "include_in_gallery": True,
                    "in": {"column_A": "col_a", "column_B": "col_b", "mostly": 0.6},
                    "out": {
                        "success": True,
                    },
                },
                {
                    "title": "basic_negative_test",
                    "exact_match_out": False,
                    "include_in_gallery": True,
                    "in": {"column_A": "col_a", "column_B": "col_b", "mostly": 1},
                    "out": {
                        "success": False,
                    },
                },
            ],
        }
    ]
    # Setting necessary computation metric dependencies and defining kwargs, as well as assigning kwargs default values
    success_keys = (
        "column_A",
        "column_B",
        "mostly",
    )

    default_kwarg_values = {
        "row_condition": None,
        "condition_parser": None,  # we expect this to be explicitly set whenever a row_condition is passed
        "mostly": 1.0,
        "result_format": "COMPLETE",
        "include_config": True,
        "catch_exceptions": False,
    }
    args_keys = (
        "column_A",
        "column_B",
    )

    def validate_configuration(
        self, configuration: Optional[ExpectationConfiguration]
    ) -> None:
        super().validate_configuration(configuration)
        if configuration is None:
            configuration = self.configuration
        try:
            assert (
                "column_A" in configuration.kwargs
                and "column_B" in configuration.kwargs
            ), "both columns must be provided"
        except AssertionError as e:
            raise InvalidExpectationConfigurationError(str(e))

    # This dictionary contains metadata for display in the public gallery
    library_metadata = {
        "tags": [],
        "contributors": ["<YOUR GITHUB USERNAME HERE>"],
    }


if __name__ == "__main__":
    ExpectProportionalFloorDifference().print_diagnostic_checklist()
# Note to users: code below this line is only for integration testing -- ignore!

diagnostics = ExpectProportionalFloorDifference().run_diagnostics()

for check in diagnostics["tests"]:
    assert check["test_passed"] is True
    assert check["error_diagnostics"] is None

for check in diagnostics["errors"]:
    assert check is None

for check in diagnostics["maturity_checklist"]["experimental"]:
    if check["message"] == "Passes all linting checks":
        continue
    assert check["passed"] is True
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

### Exercise 2
```
def feat_eng_all_steps(X: pd.DataFrame, y: pd.Series) -> (pd.DataFrame, pd.Series, Optional[pd.DataFrame]):
    try:
        return feat_eng_step_1(X), y
    except pa.errors.SchemaError as e:
        isolated_failure_cases = e.failure_cases
        # Send isolated_failure_cases to a separate function to handle them
        return X[~X.index.isin(isolated_failure_cases["index"])], y[~y.index.isin(isolated_failure_cases["index"])], isolated_failure_cases
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