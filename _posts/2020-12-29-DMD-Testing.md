---
layout: post
title: "Testing the DMD Algorithm"
---

### Goal and Overview
In this exercise, I would like to create a toy problem to test the Dynamic Mode Decomposition (DMD) algorithm. In particular, I will first use a perfect 2D dynamical system to generate some noisy 3D data. I will then feed the noisy 3D data into DMD and see if I can recover the true 2D dynamics. The code below is written in MATLAB. I created the toy problem myself and have modified the original DMD codes in ref 1 to make it work for the toy problem.

<!--more-->
### Data Creation
In this session, I would like to create some 3D noisy data for a perfect 2D dynamical system.

<pre>
  <code class="matlab">
    % create data
    t = 0:0.01:5;
    z1 = 2.0 * cos(2.0*pi*t);
    z2 = 1.0 * sin(2.0*pi*t);
    Z = [z1; z2];
    %plot(z1,z2); hold on;
    
    W = [sqrt(2.0)/2.0, -sqrt(2.0)/2.0; sqrt(2.0)/2.0, sqrt(2.0)/2.0];
    Y = W*Z;
    y1 = Y(1,:);
    y2 = Y(2,:);
    %plot(y1,y2); hold on;
    
    X = [Y; zeros(1,length(y1))];
    x1 = X(1,:) + normrnd(0.0,0.01, 1,length(y1));
    x2 = X(2,:) + normrnd(0.0,0.01, 1,length(y1));
    x3 = X(3,:) + normrnd(0.0,0.01, 1,length(y1));
    X = [x1;x2;x3];
    %figure; plot3(x1,x2,x3); hold on; axis equal; grid on;
    
    Xp = X(:,2:end);
    X = X(:,1:end-1);
  </code>
</pre>
In the MATLAB code above, the variable $z$ represents a perfect 2D dynamics. The variable $y$ is a rotation of $z$, which is then embedded into the 3D variable $x$. Finally, some noise is added to $x$ to corrupt the signal.

### Step 1: Find the Reduced Order Space
In this session, we will use SVD to find a space with reduced order ($r=2$ is assumed to know here, but ref. 1 gives procedure to determine the optimal order).

<pre>
  <code class="matlab">
    [U, Sigma, V] = svd(X,'econ');
    r = 2;
    Ur = U(:,1:r);
    Sigmar = Sigma(1:r,1:r);
    Vr = V(:,1:r);
  </code>
</pre>
If you check the matrix $\textbf{U}$, you will find SVD correctly get the reduced state space for us.

### Step 2: Find the Dynamics in the Reduced Order Space
<pre>
  <code class='matlab'>
  Atilde = Ur'*Xp*Vr/Sigmar;
  [West, Lambda] = schur(Atilde);
  </code>
</pre>
Here $\tilde{A}$ represents the linear dynamics in the reduced order space. The Schur decomposition helps us find the eigen vectors and values for this 2D dynamics.

### Step 3: Recover the 3D Signal Using the 2D Dynamics
<pre>
  <code class='matlab'>
  Phi = Xp*(Vr/Sigmar)*West;
  alpha1 = Sigmar*Vr(1,:)';
  b = (West*Lambda)\alpha1;
  
  nc = 100;
  Xest = zeros(3,nc);
  for i=1:1:100
    Xest(:,i) = Phi*(Lambda^(i-1))*b;
  end
  </code>
</pre>

### Step 4: Plotting the Results
<pre>
  <code class='matlab'>
  figure;
  plot3(X(1,:),X(2,:),X(3,:),'bo'); hold on;
  plot3(Xest(1,:),Xest(2,:),Xest(3,:),'k-','LineWidth',2.0); hold on;
  </code>
</pre>
If you run the entire code, you will find we succesfully recover a smooth 3D curve using a learnt 2D dynamics. Noise is removed and the 3D data set is digested into a 2D dynamical system.

### Reference
1.Brunton, S. L. & Kutz, J. N. Data-Driven Science and Engineering: Machine Learning, Dynamical Systems, and Control. (Cambridge University Press, 2019). doi:10.1017/9781108380690.