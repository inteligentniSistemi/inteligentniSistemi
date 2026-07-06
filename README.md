( 2*(u(1)-minulaz)/(maxulaz-minulaz)-1)
((u(1)+1)*(maxizlaz-minizlaz)/2+minizlaz)

P=[y(:,1)]; 
T=[y(:,2)]; 
vel=length(P); 
minulaz=min(P);      
maxulaz=max(P); 
minizlaz=min(T);     
maxizlaz=max(T); 
p=2*(P-minulaz)./(maxulaz - minulaz) - 1;     
% normiranje ulaza 
t=2*(T-minizlaz)./(maxizlaz - minizlaz) - 1;  % normiranje izlaza 
net=newff([-1 1],[60 1],{'tansig','purelin'},'trainlm'); 
net.trainParam.epochs=1000; 
net.trainParam.show=100; 
net.trainParam.time=Inf; 
net.trainParam.goal=2e-9; 
net.performFcn='sse'; 
tic 
net=train(net,p',t'); 
toc 
izlaz=sim(net,p') 
izlaz=(izlaz+1)*(maxizlaz-minizlaz)./2 +minizlaz; 
figure 
plot(P',izlaz,'r') 
title('Podaci dobiveni sa treniranom mrezom'); 
xlabel('Uzorci') 
ylabel('Amplituda') 
gensim(net) 



OCJENI BUTTON
% Ocitaj 5 ulaza
cijena    = app.CijenaEditField.Value;
stanje    = app.StanjeEditField.Value;
potrosnja = app.PotrosnjaEditField.Value;
ubrzanje  = app.UbraznjeEditField.Value;
godiste   = app.GodisteEditField.Value;

% 1. FUZZY sistem
fis = readfis('KupovinaMotora');
izlaz_fuzzy = evalfis(fis, [cijena stanje potrosnja ubrzanje godiste]);

% 2. Posalji ulaze u base workspace (za Simulink)
assignin('base','cijena',cijena);
assignin('base','stanje',stanje);
assignin('base','potrosnja',potrosnja);
assignin('base','ubrzanje',ubrzanje);
assignin('base','godiste',godiste);

% 3. Pokreni Simulink EkspertniSistem
load_system('EkspertniSistem');
set_param('EkspertniSistem','SimulationCommand','start');
pause(1);

izlaz_ann = evalin('base','izlazSim1');  % ANN
izlaz_es  = evalin('base','izlazSim');   % Ekspertni

% 4. Prikazi brojeve
app.FuzzyEditField.Value     = izlaz_fuzzy(1);
app.NNEditField.Value        = izlaz_ann(1);
app.EkspretniEditField.Value = izlaz_es(1);



PONISTI
app.CijenaEditField.Value = 0;
app.StanjeEditField.Value = 0;
app.PotrosnjaEditField.Value = 0;
app.UbraznjeEditField.Value = 0;
app.GodisteEditField.Value = 0;
app.FuzzyEditField.Value = 0;
app.NNEditField.Value = 0;
app.EkspretniEditField.Value = 0;




IZADJI
delete(app);
