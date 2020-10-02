POEM ID: 032  
Title: Detailed Driver Scaling Report  
authors: [justinsgray]    
Competing POEMs: N/A   
Related POEMs: N/A  
Associated implementation PR:  

Status:  

- [ ] Active  
- [ ] Requesting decision  
- [x] Accepted  
- [ ] Rejected  
- [ ] Integrated  



Motivation
----------
Getting scaling right on a large optimziation is challenging, 
even when you have a good sense of the problem. 
Most users --- especially ones who are debugging other peoples opts ---
don't have a complete picture of all the DVs, objectives, and constraints. 

The framework should be able to produce a report showing the model values and the scaled values (i.e. as the drive sees them) for all DVs, objectives, constraints. 


Description
-----------

This new feature should be accessible both from a method the user can call on driver, 
or via the OpenMDAO command line. 

Because the scaling report will include information about the objective, constraints, 
and potentially derivatives it will need to be run before generating the report. 
So the driver method will have to call run_driver once. 
This call to run_driver should not trigger any case recording (this might be challenging...)


Consider a problem with 15 design variables, 10 constraints, and one objective. 
Users have provided values for ref, ref0, upper and lower (for dvs) for some or all of these.
We need to give the user a detailed summary of the problem scaling. 
Keeping in mind that some or all of the dvs, objectives, 
and constraints might be scalar or arrays. 


The scaling report should be formatted like this: 

```
Design Variables
-----------------

                model  driver                 model driver    model  driver 
 name (shape) | value  (value) | ref | ref0 | lower (lower) | upper (upper) | 
----------------------------------------------------------------------------
x (1)           10      (1)      10     N/A     -20   (-0.2)   100    (10)    
y (10)         |31.6|   (|31.6|)  1     N/A     -100  (-100)   100    (100)    

```

Notes: 

- Array values should be shown as the 2-norm of the array
- Its likely that  the value will be an array, but the ref/ref0 or upper/lower will be scalar. 
  only the array values should be shown as a norm. 
- There will be an option for print-arrays, in which case arrays are flattened and expanded vertically. 
  even though the variable is flattened to show, the shape note should contain the correct shape 
- There will be an option to print the min/max values for arrays (rather than show the full array). 
  The associated ref/ref0 upper/lower will match up with the index of the min/max
- There will be separate sections for design variables, objectives, constraints. 
- The constraint section will add a column idenifying the type (upper/lower/equal). 
  It will also have columns for the current value, along with the constraint value shown in both model and driver scaled values. 
- If a constraint is specified with both lower and upper, then separate rows will output for each argument. 
- At the start of each section should be a summary table which gives the minimum and maximum value based on driver scaled quantities. 


The format when arrays are fully expanded will look like this
```
Design Variables
-----------------

                model  driver                 model driver    model  driver 
 name (size)  | value  (value) | ref | ref0 | lower (lower) | upper (upper) | 
----------------------------------------------------------------------------
x (1)           10      (1)      10     N/A     -20   (-0.2)   100    (10)    
y (10)          10      (10)      1     N/A     -100  (-100)   100    (100)    
                10      (10)      1     N/A     -100  (-100)   100    (100)    
                10      (10)      1     N/A     -100  (-100)   100    (100)    
                10      (10)      1     N/A     -100  (-100)   100    (100)    
                10      (10)      1     N/A     -100  (-100)   100    (100)    
                10      (10)      1     N/A     -100  (-100)   100    (100)    
                10      (10)      1     N/A     -100  (-100)   100    (100)    
                10      (10)      1     N/A     -100  (-100)   100    (100)    
                10      (10)      1     N/A     -100  (-100)   100    (100)    
                10      (10)      1     N/A     -100  (-100)   100    (100)    

```


The format when arrays are shown as min/max
```
Design Variables
-----------------

                model  driver                 model driver    model  driver 
 name (size)  | value  (value) | ref | ref0 | lower (lower) | upper (upper) | 
----------------------------------------------------------------------------
x (1)           10      (1)      10     N/A     -20   (-0.2)   100    (10)    
y (10)         |31.6|   (|31.6|)  1     N/A     -100  (-100)   100    (100)    
  min           10      (10)      1     N/A     -100  (-100)   100    (100)    
  max           1000    (100)     1     N/A     -100  (-100)   100    (100)    
```

In addition to the display of the values, There will be an option to include derivative information as a separate section. 
The derivative section will be formatted like this: 

```
Jacobian
-----------------

Jacobian        |  Scaled Values  | 
 (of)   | (wrt) |   min   |  max  | 
----------------------------------------------------------------------------
hop0.phases.main_phase.collocation_constraint.defects:p | hop0.phases.main_phase.polynomial_control_group.indep_polynomial_controls.polynomial_controls:Thrust_R | -3898.85806659874 | 17986.579402026982 |






Jacobian (Promoted Names)
-----------------

Jacobian        |  Scaled Values  | 
 (of)   | (wrt) |   min   |  max  | 
----------------------------------------------------------------------------
 p | Thrust_R | -3898.85806659874 | 17986.579402026982 |



```

Notes: 
    - Since jacobians are very often matricies, the default should be the min-max view. 
    - The user can request a full printing, in which case the jacobian is flattened and expanded into extra rows. 
      The index of value in the sub-jac will be given in the shape column.
    - One of the challenges of using the full names for the Jacobian is they are very long, we might want to have 
      the option to display promoted names by default.
    - Additional Jacobian option to hide zero values or hide values below the coloring thrshold. This could be 
      useful to reduce unncessary output text. 
    - It would be useful to be able to display this jacobian information for every step of the optimizer. This 
      will allow the operator to determine how effective the scaling is throughout the optimization. 