# D’kitty

날짜: 2024년 9월 6일 → 2024년 9월 10일

아래의 파일을 `Download` 폴더에 받는다.

[mujoco210-linux-x86_64.tar.gz](D%E2%80%99kitty%20e461f59391da4627949e311853f06fa3/mujoco210-linux-x86_64.tar.gz)

```python

conda create -n dkitty python==3.8
pip3 install -U 'mujoco-py<2.2,>=2.1'
pip install mujoco
pip install "cython<3"
sudo apt-get install patchelf
cd ~/Download

tar -zxvf mujoco210-linux-x86_64.tar.gz 

mkdir ~/.mujoco
cp -r mujoco210 ~/.mujoco/
# 실행 테스트
cd ~/.mujoco/mujoco210/bin
./simulate ../model/humanoid.xml # -> 휴머노이드가 쓰러지는 것을 확인할 수 있다.

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/songwoo/.mujoco/mujoco210/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/nvidia
export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libGLEW.so

cd ~/.mujoco/mujoco210/model/
git clone https://github.com/vikashplus/robel_sim.git

```

```
📦.mujoco
 ┗ 📂mujoco210
 ┃ ┣ 📂bin
 ┃ ┃ ┣ 📜simulate
 ┃ ┣ 📂include
 ┃ ┣ 📂model
 ┃ ┃ ┣ 📂robel_sim
 ┃ ┃ ┃ ┗ 📂dkitty
 ┃ ┃ ┃ ┃ ┗ 📜kitty-v2.1.xml
 ┃ ┃ ┣ 📜humanoid.xml
 ┃ ┣ 📂sample
 ┃ ┗ 📜THIRD_PARTY_NOTICES
```

- 예제 코드 1
    
    ```python
    # ref : https://github.com/openai/mujoco-py/blob/master/examples/body_interaction.py
    # ref2 : https://www.freecodecamp.org/news/how-to-set-an-environment-variable-in-linux/ (set path Variable export..)
    # error -> Missing GL GLEW  
    # pip freeze | xargs pip uninstall -y
    # >> https://talkingaboutme.tistory.com/entry/RL-mujocopy-ERROR-GLEW-initalization-error-Missing-GL-version
    
    import mujoco_py
    import os
    
    from threading import Lock
    # reload_package(mujoco_py)
    mj_path = mujoco_py.utils.discover_mujoco() # ./bin/simulate ./model/humanoid.xml
    mj_path = mj_path+'/model/humanoid.xml'
    xml_path = os.path.join(mj_path)
    model = mujoco_py.load_model_from_path(xml_path)
    
    sim = mujoco_py.MjSim(model)
    viewer = mujoco_py.MjViewer(sim)
    t= 0
    while True :
        sim.step()
        viewer.render()
    viewer.close()
    ```
    
    ![image.png](D%E2%80%99kitty%20e461f59391da4627949e311853f06fa3/image.png)
    
- 예제 코드 2
    
    ```python
    # https://github.com/openai/mujoco-py/blob/master/examples/body_interaction.py
    # error -> Missing GL GLEW
    import mujoco_py
    import math
    import os
    
    MODEL_XML = """
    <?xml version="1.0" ?>
    <mujoco>
        <option timestep="0.005" />
        <worldbody>
            <body name="robot" pos="0 0 1.2">
                <joint axis="1 0 0" damping="0.1" name="slide0" pos="0 0 0" type="slide"/>
                <joint axis="0 1 0" damping="0.1" name="slide1" pos="0 0 0" type="slide"/>
                <joint axis="0 0 1" damping="1" name="slide2" pos="0 0 0" type="slide"/>
                <geom mass="1.0" pos="0 0 0" rgba="1 0 0 1" size="0.15" type="sphere"/>
    			<camera euler="0 0 0" fovy="40" name="rgb" pos="0 0 2.5"></camera>
            </body>
            <body mocap="true" name="mocap" pos="0.5 0.5 0.5">
    			<geom conaffinity="0" contype="0" pos="0 0 0" rgba="1.0 1.0 1.0 0.5" size="0.1 0.1 0.1" type="box"></geom>
    			<geom conaffinity="0" contype="0" pos="0 0 0" rgba="1.0 1.0 1.0 0.5" size="0.2 0.2 0.05" type="box"></geom>
    		</body>
            <body name="cylinder" pos="0.1 0.1 0.2">
                <geom mass="1" size="0.15 0.15" type="cylinder"/>
                <joint axis="1 0 0" name="cylinder:slidex" type="slide"/>
                <joint axis="0 1 0" name="cylinder:slidey" type="slide"/>
            </body>
            <body name="box" pos="-0.8 0 0.2">
                <geom mass="0.1" size="0.15 0.15 0.15" type="box"/>
            </body>
            <body name="floor" pos="0 0 0.025">
                <geom condim="3" size="1.0 1.0 0.02" rgba="0 1 0 1" type="box"/>
            </body>
        </worldbody>
        <actuator>
            <motor gear="2000.0" joint="slide0"/>
            <motor gear="2000.0" joint="slide1"/>
        </actuator>
    </mujoco>
    """
    
    model = mujoco_py.load_model_from_xml(MODEL_XML)
    sim = mujoco_py.MjSim(model)
    viewer = mujoco_py.MjViewer(sim)
    t= 0
    while True:
        sim.data.ctrl[0] = math.cos(t / 10.) * 0.01
        sim.data.ctrl[1] = math.sin(t / 10.) * 0.01
        t += 1
        sim.step()
        viewer.render()
        if t > 100 and os.getenv('TESTING') is not None:
            break
    #print(sim.data.qpos)
    ```
    
    ![image.png](D%E2%80%99kitty%20e461f59391da4627949e311853f06fa3/image%201.png)
    
- 예제 코드 3 (D’kitty)
    
    하기 파일을  `~/.mujoco/mujoco210/model/robel_sim/dkitty/` 폴더에 넣는다.
    (또는 파일 내용을 복붙해도 된다.)
    
    [kitty-v2.1.xml](D%E2%80%99kitty%20e461f59391da4627949e311853f06fa3/kitty-v2.1.xml)
    
    ```python
    # ref1 : https://mujoco.readthedocs.io/en/2.2.0/programming.html  (mujoco222 install)
    # ref2 : https://github.com/vikashplus/robel_sim/tree/9d774735f5d6a599774d01d8808f411054f8cf28/dclaw (dclaw mujoco200)
    # ref3 : export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libGLEW.so   << add must be add
    import mujoco_py
    import math
    import os
    """
        - TAB: Switch between MuJoCo cameras.
        - H: Toggle hiding all GUI components.
        - SPACE: Pause/unpause the simulation.
        - RIGHT: Advance simulation by one step.
        - V: Start/stop video recording.
        - T: Capture screenshot.
        - I: Drop into ``ipdb`` debugger.
        - S/F: Decrease/Increase simulation playback speed.
        - C: Toggle visualization of contact forces (off by default).
        - D: Enable/disable frame skipping when rendering lags behind real time.
        - R: Toggle transparency of geoms.
        - M: Toggle display of mocap bodies.
        - 0-4: Toggle display of geomgroups
    """
    mj_path = mujoco_py.utils.discover_mujoco()
    # mj_path = mj_path+'/model/robel_sim/dclaw/dclaw3xh.xml'
    mj_path = mj_path+'/model/robel_sim/dkitty/kitty-v2.1.xml'
    xml_path = os.path.join(mj_path)
    # '/home/songwoo/.mujoco222/mujoco210/model/robel_sim/dclaw/dclaw3xh.xml'   => install mujoco210
    model = mujoco_py.load_model_from_path(xml_path)
    sim = mujoco_py.MjSim(model)
    viewer = mujoco_py.MjViewer(sim)
    t= 0
    
    # these ID must be remapped to real dynamixel ID
    # ctrl value's unit is degree and dynamixel rotate 2000 -> = 0 degree
    while True:  
        # sim.data.ctrl[0] = math.cos(t / 10.) * 1   # actuator ID 10 
        sim.data.ctrl[1] = 90   # actuator ID 11``
        # sim.data.ctrl[2] = 0.9   # actuator ID 12
        # sim.data.ctrl[3] = 0.1 # actuator ID 20  (actuactor degree which is abs(45) bounded by .mjcf)
        # sim.data.ctrl[4] = math.cos(t / 10.) * 0.1 # actuator ID 21
        sim.data.ctrl[5] = math.sin(t / 100.) * 1.5 # actuator ID 22
        # sim.data.ctrl[6] = math.cos(t / 10.) * 0.1 # actuator ID 30
        # sim.data.ctrl[7] = math.sin(t / 10.) * 0.1 # actuator ID 31
        # sim.data.ctrl[8] = math.sin(t / 10.) * 0.1 # actuator ID 32
        t += 1
        sim.step(t)
        viewer.render()
        if t > 100 and os.getenv('TESTING') is not None:
            break
    #print(sim.data.qpos)
    ```
    
    ![하면 발가락 까딱까딱한다.](D%E2%80%99kitty%20e461f59391da4627949e311853f06fa3/image%202.png)
    
    하면 발가락 까딱까딱한다.
    

![image.png](D%E2%80%99kitty%20e461f59391da4627949e311853f06fa3/image%203.png)

![image.png](D%E2%80%99kitty%20e461f59391da4627949e311853f06fa3/image%204.png)
