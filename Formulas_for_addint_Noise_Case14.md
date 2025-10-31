This script adds Gaussian noise with zero mean to different power system quantities for the IEEE 14-bus case. The goal
is to simulate more realistic measurement data rather than using idealized, noise-free values. The noise standard
deviations are based on established standards and literature to ensure realism in the simulated measurements [34]-[37].

[34] Measuring Relays and Protection Equipment–Part 118-1: Synchrophasor for Power Systems–Measurements, IEEE/IEC
60255-118-1:2018 Std., Dec. 2018.  

[35] G. Valverde, S. Chakrabarti, E. Kyriakides et al., “A constrained formulation for hybrid state estimation,” IEEE
Transactions on Power Systems, vol. 26, no. 3, pp. 1102–1109, Aug. 2011.  

[36] I. Džafić, R. A. Jabr, and T. Hrnjić, “Hybrid state estimation in complex variables,” IEEE Transactions on Power
Systems, vol. 33, no. 5, pp. 5288–5296, Sep. 2018.  

[37] I. Džafić and R. A. Jabr, “Real-time equality-constrained hybrid state estimation in complex variables,”
International Journal of Electrical Power & Energy Systems, vol. 117, p. 105634, May 2020.  

```matlab
%% for power flows: std = 0.02pu
mu_pf = 0;
std_pf = 0.02;
a = readmatrix('~/Your_Path/Real_Power_From_case14.csv'); % a is the original matrix
S_base = 100;
a_pu = a/S_base; % S_bas = 100MVA (for case6) and Pd and Qd are given in MW and MVAR
b = a_pu + a_pu.*normrnd(mu_pf,std_pf,size(a_pu)); % b = a + gaussian (normal) noise
writematrix(b, '~/Your_Path/With_Noise_Real_Power_From_case14.csv');

z = readmatrix('~/Your_Path/Reactive_Power_From_case14.csv');
z_pu = z/S_base;
t = z_pu + z_pu.*normrnd(mu_pf,std_pf,size(z_pu));
writematrix(t, '~/Your_Path/With_Noise_Reactive_Power_From_case14.csv');

%% for power injection: std = 0.02 pu
mu_pi = 0;
std_pi = 0.02;
c = readmatrix('~/Your_Path/Real_P_Injections_case14.csv');
c_pu = c/S_base;
d = c_pu + c_pu.*normrnd(mu_pi,std_pi,size(c_pu));
writematrix(d, '~/Your_Path/With_Noise_Real_P_Injections_case14.csv');

v = readmatrix('~/Your_Path/Reactive_P_Injections_case14.csv');
v_pu = v/S_base;
u = v_pu + v_pu.*normrnd(mu_pi,std_pi,size(v_pu));
writematrix(u, '~/Your_Path/With_Noise_Reactive_P_Injections_case14.csv');

%% for voltages: std = 0.005 pu
% values already given in pu
mu_vm = 0;
std_vm = 0.005;
e = readmatrix('~/Your_Path/Voltage_Magnitudes_case14.csv');
f = e + e.*normrnd(mu_vm,std_vm,size(e));
writematrix(f, '~/Your_Path/With_Noise_Voltage_Magnitudes_case14.csv');

%% for angles: std = 0.1 degrees
% Get the Sine of the Angles
mu_theta = 0;
std_theta = deg2rad(0.1);
w = readmatrix('~/Your_Path/Voltage_Angles_case14.csv');
g = deg2rad(w);
h = g + g.*normrnd(mu_theta,std_theta,size(g));
m = sin(h);
writematrix(m,'~/Your_Path/With_Noise_Sine_of_Angles_case14.csv');

w = readmatrix('~/Your_Path/Branch_Current_Angles_From_case14.csv');
g = deg2rad(w);
h = g + g.*normrnd(mu_theta,std_theta,size(g));
m = sin(h);
writematrix(m,'~/Your_Path/With_Noise_Branch_Current_Angles_From_case14.csv');

w = readmatrix('~/Your_Path/Branch_Current_Angles_To_case14.csv');
g = deg2rad(w);
h = g + g.*normrnd(mu_theta,std_theta,size(g));
m = sin(h);
writematrix(m,'~/Your_Path/With_Noise_Branch_Current_Angles_To_case14.csv');

%% for branch currents: std = 0.005 pu
mu_vm = 0; % same as the voltage
std_vm = 0.005;
e = readmatrix('~/Your_Path/Branch_Current_Magnitudes_From_case14.csv');
I_base = 251.65;
I_pu = e/I_base;
f = I_pu + I_pu.*normrnd(mu_vm,std_vm,size(e));
writematrix(f, '~/Your_Path/With_Noise_Branch_Current_Magnitudes_From_case14.csv');

e = readmatrix('~/Your_Path/Branch_Current_Magnitudes_To_case14.csv');
I_pu = e/I_base;
f = I_pu + I_pu.*normrnd(mu_vm,std_vm,size(e));
writematrix(f, '~/Your_Path/With_Noise_Branch_Current_Magnitudes_To_case14.csv');
```
