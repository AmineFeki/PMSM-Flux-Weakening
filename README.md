# PMSM-Flux-Weakening

In some applications, we need a high high motor speed, even higher than the rated speed of the motor. Here comes the role of the Flux Weakening Technique.

Before carrying on disovering this project, I invite you to read carefully the document "" that I have prepared by myself.

After reading the document, let's attack the practical study!

# 1- Highlight the need of the FW technique:

Let’s assume that in a given application, we need an angular speed equal to 7000 rpm.

We command the motor with speed regulation using the command ```MC_ProgramSpeedRampMotor1_F(7000, 500);```

We obtain the following result: 
![image](https://user-images.githubusercontent.com/53936812/183295509-5e7b18c2-0142-4df6-b42b-a1e7c5f8c103.png)

![image](https://user-images.githubusercontent.com/53936812/183295523-5ac05d37-9aba-4566-8982-c05932e4ded4.png)

## Interpretation:
• Figure 1 shows that the PMSM is unable to reach the command speed. It is not dependent
on the PID constants of the speed regulator. Here, the PMSM has reached the maximum
speed with the rated voltage (in the maximum torque per ampere (MTPA) condition for a
speed lower than the rated speed).

• Figure 2 shows that the PMSM never reaches the command current. It implies that it never
reaches the torque command. This phenomenon is due to the non-need of such a torque as
the motor is not training a heavy load that requires an important torque.

• Figure 2 shows also that the Id_ref = 0. This represents the MTPA condition for a speed
lower than the rated speed.

In these conditions, the PMSM will never reach the desired value. However, the current
command is much higher than the needed value.

==> The FW technique is the solution here. It is based on losing flux and torque in favor of
increasing the speed response. It matches our needs.

# 2- Implement the FW technique based on ST firmware:

First of all, we need to configure the additional features of the algorithm and check the Flux
weakening and generate code.

![image](https://user-images.githubusercontent.com/53936812/183295694-d60ff6ad-8569-4599-a1f9-58d750657eae.png)


For more simplification, flux weakening is not based on exact formulas. It is based on PID
controller.

It is implemented as follows:

• We all know the following basic bloc diagram describing the FOC algorithm.

Iq-ref is the output of a PI controller.

Id-ref is set to zero. This condition represents the MTPA condition.

![image](https://user-images.githubusercontent.com/53936812/183295725-bb9b895d-a1dd-4826-8ab7-651f333e46b9.png)

It is obvious that the PI controller output is saturated by a saturator to protect the motor. So,
the bloc diagram in the red circle, is following one.

![image](https://user-images.githubusercontent.com/53936812/183295732-62a27b5f-33ca-46f7-b54a-e11ea413800a.png)

The flux weakening technique acts on this part of the bloc diagram as follows:
![image](https://user-images.githubusercontent.com/53936812/183295761-58ed91f8-25d8-4b65-9a3c-87a468ebee90.png)

(*) bloc describes the following formula: ![image](https://user-images.githubusercontent.com/53936812/183295788-6bf06bb4-4d3d-4b34-90e8-256cd0afc9dc.png)

As mentioned before, the idea here is sacrifice flux and torque by obtaining a negative Id
current. 
We must bear attention! ![image](https://user-images.githubusercontent.com/53936812/183295796-f11f06e2-d24e-452b-8e21-6a8c8203c8b1.png)

When Id-ref < 0 ➔ Iq-ref decreases. We, obviously, lose torque.

# Code:

After generating code, we should set the PI constants of the FW regulator. We use the
following code.

```
/* Set FW PID constants BEGIN */
PID_SetKP(&PIDFluxWeakeningHandle_M1, 400);
PID_SetKI(&PIDFluxWeakeningHandle_M1, 50);
/* Set FW PID constants END */
```

And we run the code.

# 3- Results and interpretation:

After running the code, we obtain the following result.

![image](https://user-images.githubusercontent.com/53936812/183295923-c417fe70-d6e4-47cc-a4db-f4378289a2c8.png)

We can easily identify the effect of the flux weakening technique. It allows us reach the
desired speed.

![image](https://user-images.githubusercontent.com/53936812/183295947-4aeaa5fb-7f10-413c-a2ac-9da9b0f246de.png)


Figure 7 shows the impact of forcing Id-ref < 0.
It shows also Iq-ref decreases, and it has not a negative effect as the application does not
require a high torque.
We have sacrificed torque in favor of the speed.

