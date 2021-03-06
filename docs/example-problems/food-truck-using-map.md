# Food Truck Using Map

```fsharp
module Flips.Examples.FoodTruckMapExample

open Flips
open Flips.Types

// Declare the parameters for our model
let items = ["Hamburger"; "HotDog"]
let profit = Map.ofList [("Hamburger", 1.50); ("HotDog", 1.20)]
let maxIngredients = Map.ofList [("Hamburger", 300.0); ("HotDog", 200.0)]
let itemWeight = Map.ofList [("Hamburger", 0.5); ("HotDog", 0.4)]
let maxTruckWeight = 200.0

// Create Decision Variable Map<string,Decision> to represent how much of each item we should pack
// with a Lower Bound of 0.0 and an Upper Bound of Infinity
let numberOfItem =
    [for item in items do
        item, Decision.createContinuous (sprintf "NumberOf%s" item) 0.0 infinity]
    |> Map.ofList

// Create the Linear Expression for the objective
let objectiveExpression = List.sum [for item in items -> profit.[item] * numberOfItem.[item]]

// Create an Objective with the name "MaximizeRevenue" the goal of Maximizing
// the Objective Expression
let objective = Objective.create "MaximizeRevenue" Maximize objectiveExpression

// Create a Max Item Constraints
let maxItemConstraints =
    [for item in items ->
        Constraint.create (sprintf "MaxOf%s" item) (numberOfItem.[item] <== maxIngredients.[item])]

// Create a Constraint for the Max combined weight of Hamburgers and Hotdogs
let weightExpression = List.sum [for item in items -> itemWeight.[item] * numberOfItem.[item]]
let maxWeight = Constraint.create "MaxWeight" (weightExpression<== maxTruckWeight)

// Create a Model type and pipe it through the addition of the constraints
let model =
    Model.create objective
    |> Model.addConstraints maxItemConstraints
    |> Model.addConstraint maxWeight

// Call the `solve` function in the Solve module to evaluate the model
// using the basic settings
let result = Solver.solve Settings.basic model

printfn "-- Result --"

// Match the result of the call to solve
// If the model could not be solved it will return a `Suboptimal` case with a message as to why
// If the model could be solved, it will print the value of the Objective Function and the
// values for the Decision Variables
match result with
| Optimal solution ->
    printfn "Objective Value: %f" (Objective.evaluate solution objective)

    for (decision, value) in solution.DecisionResults |> Map.toSeq do
        let (DecisionName name) = decision.Name
        printfn "Decision: %s\tValue: %f" name value
| _ -> printfn $"Unable to solve. Error: %A{result}"
```