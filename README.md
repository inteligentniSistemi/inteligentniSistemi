Expression ( 2*(u(1)-minulaz)/(maxulaz-minulaz)-1)
 Expression ((u(1)+1)*(maxizlaz-minizlaz)/2+minizlaz)

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
