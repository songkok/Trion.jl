# Trion
This is a Julia numerical package for calculating the spectrum and wavefunction of exciton and trion in two-dimensional materials.

## Tutorial
This package includes two files. "Trion.jl" is the main program for calculating the exciton/trion boundstate properties. "Nonlinearity.jl" is estimate the nonlinearity arising from exciton-exction and trion-trion interactions. To use them

### One exciton/trion boundstate calculation ("Trion.jl")

To calculation the boundstate, 
```julia
include("Trion.jl")
```
Then, calling the main function: 'spectrum' for exciton calculation as follow
```julia
exciton = spectrum([W],[me mh],alpha,Q,N,N0,L0)
# W is the screening dielectric constant in 2D (see later text)
# me = conduction band mass, mh = valence band mass [ in the unit of free electron mass]
# alpha: 1 = ground state, 2,3,4,... = excited states
# Q = [Qx Qy] is the exciton momentum.
# N is the cutoff parameter for the basis set.
# N0 and L0 are initial guess of the boundstate solution. 
# Typicall, N0 < 3 (initial size of the basis set) and L0 is about the Bohr radius.
```
In this package, the 2D potential is defined as

$$ V(q) = \frac{1}{\epsilon_q} \frac{2\pi}{q}\quad\quad \text{  (cgs unit)}$$

where $\epsilon_q$ is a 2D momentum-dependent screening dielectric constant (dimensionless). 
We note that, $\epsilon_q=1$ in vacuum. For the commonly used Keldysh potential in 2D materials, $\epsilon_q=1+r_\ast q$ where $r_\ast$ is the screening parameter. 

For the input in the code, 
$$W=\frac{1}{\epsilon_q}.$$

Once the calculation is done. One can retrive the boundstate properties by the object 'exciton' as follow
```julia
exciton.energy  # boundstate energy
exciton.r       # electron and hole average distance [sqrt(<r^2>)] 
exciton.A       # exciton wavefunction expanded in the Hermite functions basis
```

### Example: exciton and trion in TMDC monolayer

```julia
#=== Model parameters =====================#
m1=0.38 #(1st conduction band mass [per free electron mass])
m2=0.38 #(2nd conduction band mass [per free electron mass] *for trion only)
mh=0.44 #(Valance band mass [per free electron mass])
epsilon=1          #(substrate dieletric constant)
r0=40.0/epsilon    #(screen length for monolayer Keldysh potential [Ang])
Q=[0 0]            #(Total momentum [Ang^(-1)])

r1=40.0            #(screen length for Layer 1 [Ang])
r2=45.0            #(screen length for Layer 2 [Ang])
L=6.48             #(Layer 1 and 2 interlayer distance [Ang])

N0=3    # basis size (small) for optimizing the basis length
N=9     # basis size for convergent calculation
l0=[10] # initial guess of the basis length [Ang]. input [lambda1 lambda2] will active the optimization with 2 lengths. 
alpha=1 # 1 = ground state, 2,3,4,... = excited states

#V=[W(e1<->e2), W(e1<->h), W(e2<->h)] (trion)
#V=[W(e<->h)] (exciton)
#where The default potential are:
# W=VKel (monolayer Keldysh interaction)
VKel(q)=1/(epsilon*(1+r0*q))
# W=Vintra1, Vintra2 (intralayer interactions in layer 1,2) 
Vintra1(q)=(1+r2*q*(1-exp(-2*L*q)))/((1+r1*q)*(1+r2*q)-r1*r2*q^2*exp(-2*L*q))
Vintra2(q)=(1+r1*q*(1-exp(-2*L*q)))/((1+r1*q)*(1+r2*q)-r1*r2*q^2*exp(-2*L*q))
# W= Vinter (interlayer interaction)
Vinter(q)=exp(-q*L)/((1+r1*q)*(1+r2*q)-r1*r2*q^2*exp(-2*L*q))

#=== Exiton/Trion Energy & Wavefunction =====================#
exciton=spectrum([VKel],[m1 mh],alpha,Q,N,N0,9.0)
trion=spectrum([VKel VKel VKel],[m1 m2 mh],alpha,Q,N,N0,9.0)


#=== Nonlinearity calculation ===============================#
#exciton
# gX(VKel,exciton,N,5)
#trion
# gT(VKel,trion,N,10)

```




## References:

1. [KW Song, S. Chiavazzo, I. A. Shelykh, O. Kyriienko, Attractive trion-polariton nonlinearity due to Coulomb scattering](https://arxiv.org/abs/2204.00594)
    
1. [KW Song, S. Chiavazzo, I. A. Shelykh, O. Kyriienko, Theory for Coulomb scattering of trions in 2D materials](https://arxiv.org/abs/2207.02660)

