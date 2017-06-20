# RobotArm

This problem can be found [here](http://www.gpops2.com/Examples/RobotArm.html).
 although that example is missing initial and final state constraints and limits on x4


## Packages that will be used
```@example RobotArm
using NLOptControl
nothing # hide
```

## Differential Equations
```@example RobotArm
const EP=2*eps()
function RobotArm{T<:Any}(n::NLOpt,x::Array{T,2},u::Array{T,2}) # dynamic constraint equations
  if n.s.integrationMethod==:tm; L=size(x)[1]; else; L=size(x)[1]-1; end
  dx=Array(Any,L,n.numStates);
  Q=5;
  I_t=@NLexpression(n.mdl, [j=1:L], ((Q-x[j,1])^3+x[j,1]^3)/3*sin(x[j,5])^2 );
  I_p=@NLexpression(n.mdl, [j=1:L], ((Q-x[j,1])^3+x[j,1]^3)/3 );

  dx[:,1]=@NLexpression(n.mdl, [j=1:L], x[j,2]);
  dx[:,2]=@NLexpression(n.mdl, [j=1:L], u[j,1]/Q);
  dx[:,3]=@NLexpression(n.mdl, [j=1:L], x[j,4]);
  dx[:,4]=@NLexpression(n.mdl, [j=1:L], u[j,2]/(I_t[j]+EP));
  dx[:,5]=@NLexpression(n.mdl, [j=1:L], x[j,6]);
  dx[:,6]=@NLexpression(n.mdl, [j=1:L], u[j,3]/(I_p[j]+EP));
  return dx
end
nothing # hide
```

## Define and Configure the Problem:
```@example RobotArm
n=define(RobotArm;numStates=6,numControls=3,X0=[9/2,0.0,0.0,0.0,pi/4,0.0],XF=[9/2,0.0,2*pi/3,0.0,pi/4,0.0],XL=[NaN,NaN,NaN,0.0,NaN,NaN],XU=[NaN,NaN,NaN,1.0,NaN,NaN],CL=[-1.,-1.,-1.],CU=[1.,1.,1.])
configure!(n;(:finalTimeDV=>true))
nothing # hide
```

## Objective Function
```@example RobotArm
@NLobjective(n.mdl,Min,n.tf);
nothing # hide
```

## Optimize
```@example RobotArm
optimize!(n);
nothing # hide
```

## Post Process
```@example RobotArm
using PrettyPlots
allPlots(n)
```