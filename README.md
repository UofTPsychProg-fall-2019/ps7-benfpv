# thegameplan

## results <- analysis(data)
For this problem set, your task is write out pseudocode for the analyses you'll need for your project. Think about all of the steps you'll need to transform your raw data into the central summary measures and statistical tests that will reveal your amazing findings. What summary measures do you need to answer your research question? Do you need to recode variables (e.g., recode responses into correct vs. incorrect), manage potentially missing data, or try to find outliers? Write out the logical steps in an organized and structured format. If you don't know all the details of your analysis, that is fine. Go with something that's sensible. But, the closer you are to the actual analysis you will implement, the more helpful this assignment will be later on!

## Deliverables
Your submission should be 1 text file with:
- A brief paragraph explaining the raw data from your experiment (what are your IVs and DVs?) and the summary measures you'll need to compute (e.g., accuracy, median reaction times, average confidence ratings).
- Your pseudocode that details how you'll turn raw data into findings.

[Inspiration](https://www.youtube.com/watch?v=VcjzHMhBtf0)

############################################################################

## Brief Paragraph
- Past research found that human attention can cycle at 4Hz or 8Hz when attending to stimuli.
- Here, we want to parse whether frame-by-frame feedback of accuracy helps participants track a stimulus with mouse (pilot) or eye-tracking (real).
# Task
- Begin trial ---
- Thus, we first present participants with a stimulus to track on-screen.
- The stimulus is always presented at a random place on-screen, away from the edges.
- After 500ms, the stimulus starts to accelerate spatially at a random magnitude (with maximum) in a random direction per frame. Stimulus emits a coloured (counterbalanced red, green, blue) circle from its middle to a radius (mouse distance from stimulus) to indicate random performance, immediate performance, or average performance over the most recent .5 second, 1 second, 2 seconds, or entire trial; differs by trial. Tracking lasts for a couple seconds, counterbalanced.
- All stimuli except the fixation dot disappears for a second (retention interval).
- End of trial ---
# IV and DV
- The IV is whether the stimulus is 
- The DV is participant's mean tracking accuracy per frame.
- Another fun measure is frame-by-frame frequency of participant's accuracy MINUS the frame-by-frame frequency of acceleration magnitude of the stimulus in any direction.

## Analysis
Import experiment data.
Import participant's raw mouse acceleration, velocity, and position values.
Import stimulus raw acceleration, velocity, and position values.
Import participant's accuracy (radius of the mouse from the stimulus at each frame of each trial).

Calculate participant's mean tracking accuracy per frame across all trials, for each of the conditions (average performance (distance from stimulus) over the most recent .5 second, 1 second, 2 seconds, or entire trial.)
	
Fourier transform the radius data.
	Band-pass filter participant accuracy for each frequency (Not Done!).
	Multiply the convolution by Gabor wavelet on a per-trial basis (Not Done!).
	Store the results of Fourier transform on a per-trial basis.

Code for converting x,y mouse tracking distance from stimulus and its Fourier transform are already done here:

        #%% By-trial Analysis
        # To access position data per trial:
        # d_xxx_pos[block][trial][timepoint]
        time_analysis_start=trialClock.getTime()
        trial_fps=frame_track_end/time_track_dur
        trial_tpf=time_track_dur/frame_track_end #time per frame (tpf)
        if trial_analysis==1:
            # Collapse data to 1-dimensional (i.e., radius from target).
            for current_acc in range(len(current_acc_x)):
                if current_acc==0:
                    acc_rad=[np.sqrt(current_acc_x[current_acc]**2+current_acc_y[current_acc]**2)]
                else:
                    acc_rad.append(np.sqrt(current_acc_x[current_acc]**2+current_acc_y[current_acc]**2))
            # Unidimensional Accuracy Descriptive Statistics
            acc_rad_mean=np.mean(acc_rad[1::])
            acc_rad_error=acc_rad_mean-acc_rad[1::]
            acc_rad_sqerror=acc_rad_error**2
            acc_rad_ss=sum(acc_rad_sqerror)
            acc_rad_sd=acc_rad_ss/len(acc_rad[1::])-1
            # Plot a 2d 'heatmap' of accuracy.
            for current_pix in range(len(acc_rad)):
                if current_pix==0:
                    heatmap=[psychopy.visual.Circle(win=win,pos=(current_acc_x[current_pix],current_acc_y[current_pix]),color=current_pix/len(acc_rad),colorSpace='rgb',radius=.001,edges=4)]
                else:
                    heatmap.append(psychopy.visual.Circle(win=win,pos=(current_acc_x[current_pix],current_acc_y[current_pix]),color=current_pix/len(acc_rad),colorSpace='rgb',radius=.001,edges=4))
            heatmap_mean=psychopy.visual.Circle(win=win,pos=(np.mean(current_acc_x),np.mean(current_acc_y)),color=(0,1,0),colorSpace='rgb',radius=.002,edges=4)
            #%% Fourier transform of mouse-stim accuracy throughout trial.
            fourier_freqs=np.arange(fourier_min_freq,fourier_max_freq,fourier_freq_res) #frequency in seconds of one complete fourier cycle.
            fourier_magnifier=40
            for freq in range(len(fourier_freqs)):
                if freq==0:
                    fourier_fpf=[trial_fps*fourier_freqs[freq]] #frames per fourier_freq
                    fourier_dpf=[360/fourier_fpf[freq]] #degrees per frame on 360 circular space
                else:
                    fourier_fpf.append(trial_fps*fourier_freqs[freq])
                    fourier_dpf.append(360/fourier_fpf[freq])
                # convert acc_rad into point along fourier circle.
                for acc in range(len(acc_rad)):
                    if acc==0:
                        fourier_acc_rad_x=[np.cos(fourier_dpf[freq]*acc)*acc_rad[acc]]
                        fourier_acc_rad_y=[np.sin(fourier_dpf[freq]*acc)*acc_rad[acc]]
                        fourier_acc_rad_rad=[np.sqrt(fourier_acc_rad_x[acc]**2+fourier_acc_rad_y[acc]**2)]
                        if fourier_freqs[freq]==4.0 and trial_analysis_visual==1:
                            fourier_graph=[psychopy.visual.Circle(win=win,pos=(fourier_acc_rad_x[acc]*fourier_magnifier,fourier_acc_rad_y[acc]*fourier_magnifier),color=(0,1,1),colorSpace='rgb',radius=.001,edges=4)]
                        elif fourier_freqs[freq]==8.0 and trial_analysis_visual==1:
                            fourier_graph=[psychopy.visual.Circle(win=win,pos=(fourier_acc_rad_x[acc]*fourier_magnifier,fourier_acc_rad_y[acc]*fourier_magnifier),color=(1,1,0),colorSpace='rgb',radius=.001,edges=4)]
                        elif trial_analysis_visual==1:
                            fourier_graph=[psychopy.visual.Circle(win=win,pos=(fourier_acc_rad_x[acc]*fourier_magnifier,fourier_acc_rad_y[acc]*fourier_magnifier),color=freq/len(fourier_freqs),colorSpace='rgb',radius=.001,edges=4)]
                    else:
                        fourier_acc_rad_x.append(np.cos(fourier_dpf[freq]*acc)*acc_rad[acc])
                        fourier_acc_rad_y.append(np.sin(fourier_dpf[freq]*acc)*acc_rad[acc])
                        fourier_acc_rad_rad.append(np.sqrt(fourier_acc_rad_x[acc]**2+fourier_acc_rad_y[acc]**2))
                        if fourier_freqs[freq]==4.0 and trial_analysis_visual==1:
                            fourier_graph.append(psychopy.visual.Circle(win=win,pos=(fourier_acc_rad_x[acc]*fourier_magnifier,fourier_acc_rad_y[acc]*fourier_magnifier),color=(0,1,1),colorSpace='rgb',radius=.001,edges=4))
                        elif fourier_freqs[freq]==8.0 and trial_analysis_visual==1:
                            fourier_graph.append(psychopy.visual.Circle(win=win,pos=(fourier_acc_rad_x[acc]*fourier_magnifier,fourier_acc_rad_y[acc]*fourier_magnifier),color=(1,1,0),colorSpace='rgb',radius=.001,edges=4))
                        elif trial_analysis_visual==1:
                            fourier_graph.append(psychopy.visual.Circle(win=win,pos=(fourier_acc_rad_x[acc]*fourier_magnifier,fourier_acc_rad_y[acc]*fourier_magnifier),color=freq/len(fourier_freqs),colorSpace='rgb',radius=.001,edges=4))
                if freq==0:
                    #get fourier center of mass (CoM)
                    if trial_analysis_visual==1:
                        fourier_graph_freq=[fourier_graph]
                    fourier_com_x=[np.mean(fourier_acc_rad_x)]
                    fourier_com_y=[np.mean(fourier_acc_rad_y)]
                    fourier_com_rad=[np.sqrt(fourier_com_x[freq]**2+fourier_com_y[freq]**2)]
                    # Fourier CoM Descriptive Statistics
                    fourier_com_rad_mean=[fourier_com_rad[freq]]
                    fourier_com_rad_error=[fourier_com_rad_mean[freq]-fourier_acc_rad_rad[1::]]
                    fourier_com_rad_sqerror=[fourier_com_rad_error[freq]**2]
                    fourier_com_rad_ss=[sum(fourier_com_rad_sqerror[freq])]
                    fourier_com_rad_sd=[fourier_com_rad_ss[freq]/(len(acc_rad)-1)] #not sure about these.
                    # visual Fourier com
                    if fourier_freqs[freq]==4.0 and trial_analysis_visual==1:
                        fourier_com_dot=[psychopy.visual.Circle(win=win,pos=(fourier_com_x[freq]*fourier_magnifier,fourier_com_y[freq]*fourier_magnifier),color=(0,0,1),colorSpace='rgb',radius=.003,edges=4)]
                        fourier_com_rad_graph=[psychopy.visual.Circle(win=win,pos=(-.1*fourier_magnifier*.1+freq*fourier_freq_res*.01*fourier_magnifier*.1,fourier_com_rad[freq]*fourier_magnifier),color=(0,0,1),colorSpace='rgb',radius=.002,edges=4)]
                        #fourier_com_rad_sd_pos_graph=[psychopy.visual.Circle(win=win,pos=(.2+freq*.01,fourier_com_rad_mean[freq]+fourier_com_rad_sd[freq]),color=(0,0,1),colorSpace='rgb',radius=.001,edges=4)]
                        #fourier_com_rad_sd_neg_graph=[psychopy.visual.Circle(win=win,pos=(.2+freq*.01,fourier_com_rad_mean[freq]-fourier_com_rad_sd[freq]),color=(0,0,1),colorSpace='rgb',radius=.001,edges=4)]
                    elif fourier_freqs[freq]==8.0 and trial_analysis_visual==1:
                        fourier_com_dot=[psychopy.visual.Circle(win=win,pos=(fourier_com_x[freq]*fourier_magnifier,fourier_com_y[freq]*fourier_magnifier),color=(1,0,0),colorSpace='rgb',radius=.003,edges=4)]
                        fourier_com_rad_graph=[psychopy.visual.Circle(win=win,pos=(-.1*fourier_magnifier*.1+freq*fourier_freq_res*.01*fourier_magnifier*.1,fourier_com_rad[freq]*fourier_magnifier),color=(1,0,0),colorSpace='rgb',radius=.002,edges=4)]
                        #fourier_com_rad_sd_pos_graph=[psychopy.visual.Circle(win=win,pos=(.2+freq*.01,fourier_com_rad_mean[freq]+fourier_com_rad_sd[freq]),color=(1,0,0),colorSpace='rgb',radius=.001,edges=4)]
                        #fourier_com_rad_sd_neg_graph=[psychopy.visual.Circle(win=win,pos=(.2+freq*.01,fourier_com_rad_mean[freq]-fourier_com_rad_sd[freq]),color=(1,0,0),colorSpace='rgb',radius=.001,edges=4)]
                    elif trial_analysis_visual==1:
                        fourier_com_dot=[psychopy.visual.Circle(win=win,pos=(fourier_com_x[freq]*fourier_magnifier,fourier_com_y[freq]*fourier_magnifier),color=(0,freq/len(fourier_freqs),0),colorSpace='rgb',radius=.002,edges=4)]
                        fourier_com_rad_graph=[psychopy.visual.Circle(win=win,pos=(-.1*fourier_magnifier*.1+freq*fourier_freq_res*.01*fourier_magnifier*.1,fourier_com_rad[freq]*fourier_magnifier),color=(1,1,1),colorSpace='rgb',radius=.001,edges=4)]
                        #fourier_com_rad_sd_pos_graph=[psychopy.visual.Circle(win=win,pos=(.2+freq*.01,fourier_com_rad_mean[freq]+fourier_com_rad_sd[freq]),color=(0,freq/len(fourier_freqs),0),colorSpace='rgb',radius=.001,edges=4)]
                        #fourier_com_rad_sd_neg_graph=[psychopy.visual.Circle(win=win,pos=(.2+freq*.01,fourier_com_rad_mean[freq]-fourier_com_rad_sd[freq]),color=(0,freq/len(fourier_freqs),0),colorSpace='rgb',radius=.001,edges=4)]
                else:
                    #get fourier center of mass (com)
                    if trial_analysis_visual==1:
                        fourier_graph_freq.append(fourier_graph)
                    fourier_com_x.append(np.mean(fourier_acc_rad_x))
                    fourier_com_y.append(np.mean(fourier_acc_rad_y))
                    fourier_com_rad.append(np.sqrt(fourier_com_x[freq]**2+fourier_com_y[freq]**2))
                    # Fourier CoM Descriptive Statistics
                    fourier_com_rad_mean.append(fourier_com_rad[freq])
                    fourier_com_rad_error.append(fourier_com_rad_mean[freq]-fourier_acc_rad_rad[1::])
                    fourier_com_rad_sqerror.append(fourier_com_rad_error[freq]**2)
                    fourier_com_rad_ss.append(sum(fourier_com_rad_sqerror[freq]))
                    fourier_com_rad_sd.append(fourier_com_rad_ss[freq]/(len(acc_rad)-1))
                    #visual Fourier com
                    if fourier_freqs[freq]==4.0 and trial_analysis_visual==1:
                        fourier_com_dot.append(psychopy.visual.Circle(win=win,pos=(fourier_com_x[freq]*fourier_magnifier,fourier_com_y[freq]*fourier_magnifier),color=(0,0,1),colorSpace='rgb',radius=.003,edges=4))
                        fourier_com_rad_graph.append(psychopy.visual.Circle(win=win,pos=(-.1*fourier_magnifier*.1+freq*fourier_freq_res*.01*fourier_magnifier*.1,fourier_com_rad[freq]*fourier_magnifier),color=(0,0,1),colorSpace='rgb',radius=.002,edges=4))
                        #fourier_com_rad_sd_pos_graph.append(psychopy.visual.Circle(win=win,pos=(.2+freq*.01,fourier_com_rad_mean[freq]+fourier_com_rad_sd[freq]),color=(0,0,1),colorSpace='rgb',radius=.001,edges=4))
                        #fourier_com_rad_sd_neg_graph.append(psychopy.visual.Circle(win=win,pos=(.2+freq*.01,fourier_com_rad_mean[freq]-fourier_com_rad_sd[freq]),color=(0,0,1),colorSpace='rgb',radius=.001,edges=4))
                    elif fourier_freqs[freq]==8.0 and trial_analysis_visual==1:
                        fourier_com_dot.append(psychopy.visual.Circle(win=win,pos=(fourier_com_x[freq]*fourier_magnifier,fourier_com_y[freq]*fourier_magnifier),color=(1,0,0),colorSpace='rgb',radius=.003,edges=4))
                        fourier_com_rad_graph.append(psychopy.visual.Circle(win=win,pos=(-.1*fourier_magnifier*.1+freq*fourier_freq_res*.01*fourier_magnifier*.1,fourier_com_rad[freq]*fourier_magnifier),color=(1,0,0),colorSpace='rgb',radius=.002,edges=4))
                        #fourier_com_rad_sd_pos_graph.append(psychopy.visual.Circle(win=win,pos=(.2+freq*.01,fourier_com_rad_mean[freq]+fourier_com_rad_sd[freq]),color=(1,0,0),colorSpace='rgb',radius=.001,edges=4))
                        #fourier_com_rad_sd_neg_graph.append(psychopy.visual.Circle(win=win,pos=(.2+freq*.01,fourier_com_rad_mean[freq]-fourier_com_rad_sd[freq]),color=(1,0,0),colorSpace='rgb',radius=.001,edges=4))
                    elif trial_analysis_visual==1:
                        fourier_com_dot.append(psychopy.visual.Circle(win=win,pos=(fourier_com_x[freq]*fourier_magnifier,fourier_com_y[freq]*fourier_magnifier),color=(0,freq/len(fourier_freqs),0),colorSpace='rgb',radius=.002,edges=4))
                        fourier_com_rad_graph.append(psychopy.visual.Circle(win=win,pos=(-.1*fourier_magnifier*.1+freq*fourier_freq_res*.01*fourier_magnifier*.1,fourier_com_rad[freq]*fourier_magnifier),color=(1,1,1),colorSpace='rgb',radius=.001,edges=4))
                        #fourier_com_rad_sd_pos_graph.append(psychopy.visual.Circle(win=win,pos=(.2+freq*.01,fourier_com_rad_mean[freq]+fourier_com_rad_sd[freq]),color=(0,freq/len(fourier_freqs),0),colorSpace='rgb',radius=.001,edges=4))
                        #fourier_com_rad_sd_neg_graph.append(psychopy.visual.Circle(win=win,pos=(.2+freq*.01,fourier_com_rad_mean[freq]-fourier_com_rad_sd[freq]),color=(0,freq/len(fourier_freqs),0),colorSpace='rgb',radius=.001,edges=4))
