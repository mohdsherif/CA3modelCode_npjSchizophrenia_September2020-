# Hippocampal CA3 model (Sherif et al, 2020)

This is the code for the CA3 hippocampal model in the paper by [Sherif et al](https://www.nature.com/articles/s41537-020-00109-0), 2020, in npj Schizophrenia.

The code has been updated to use Python3. In addition to [NEURON](https://neuron.yale.edu/neuron/), the following python packages are needed:
    
    hyp5
    configparser

(You can use conda or pip to install them).

After you install the required packages, compile the mod files:

    nrnivmodl

## Running one simulation (fig. 1):
To plot a figure similar to figure 1 in the paper with the control conditions, you will need first to run a control simulation. The configuration file with the parameters for the model are in fig1simulationConfig.cfg file.
The following code will run a simulation that is 3 seconds long (it takes around one minute to run on my machine with 8 cores), and save the resulting output file in directory ./data/batch.

    python runone.py
## Plotting results of one simulation (fig. 1):
After the file has been generated (or if you want to plot the provided output sample file), you can run the following code:

    python -i analysisPlottingCode.py
    
Then inside python:

    simstr = 'controlSimulation' # name of simulation that just ran - set in the configuration file.

    loadedSim = loadSimH5py(simstr, datadir = './data/batch/')
    myfig, rastsp, lfpsp, psdsp = plotloadedsimProfiling(loadedSim, 0)
    rastsp.set_xlim(2000, 3000) # plot the last 1000 ms (1 second).

    # annotate y-axis of raster for the different neuronal populations
    rastsp.set_ylabel('')
    rastsp.set_yticklabels('')
    rastsp.annotate('PYR', xy=(-0.1, 0.33), xycoords='axes fraction', color = 'red', fontsize = 12, fontweight='bold')
    rastsp.annotate('PV', xy=(-0.1, 0.73), xycoords='axes fraction', color = 'green', fontsize = 12, fontweight='bold')
    rastsp.annotate('OLM', xy=(-0.1, 0.88), xycoords='axes fraction', color = 'blue', fontsize = 12, fontweight='bold')

    # change color of LFP voltage plot to black
    lfpline = lfpsp.get_lines()[0]
    lfpline.set_color('k')

    # change color of PSD plot to black
    psdline = psdsp.get_lines()[0]
    psdline.set_color('k')

You should get a figure similar to the one below:
![Alt text](fig1sample.png?raw=true "Optional Title")

## Running a batch modifying a parameter:
If you are interested in examining variation in parameter values, e.g. scaling NMDAR conductance on OLM cells down and up (multiplying by a scaling factor), you can paste the following code into a python shell:

    from batch import batchRun, NewParam
    import os
    
    mytstop = 100
    
    lnmdarw_olm_scaling = [0.25, 0.5, 1, 1.5, 2] # scaling of 1 is the "control" where the conductance gets multiplied by 1
    
    liseed = [1234, 6912, 9876] # using 3 randomization seeds for input spike times
    lwseed = [4321, 5012, 9281] # using 3 randomization seeds for wiring
    
    
    def runBatch():
        '''run batch'''
        lsec, lopt, lval, lconfigfilestr = [], [], [], []
        for nmdarw_olm_scaling in lnmdarw_olm_scaling:
            for iseed in liseed:
                for wseed in lwseed:
                    tmpsec, tmpopt, tmpval = [], [], []
                    simstr = '2023dec06_{mytstop}_ms_olmNMDARw_{nmdarw_olm_scaling}_pvPyrIh_{ih_pvPyr_scaling}_pyrGABA_{gaba_pyr_scaling}_iseed_{iseed}_wseed_{wseed}_a'.format(mytstop = mytstop, nmdarw_olm_scaling = nmdarw_olm_scaling, ih_pvPyr_scaling = ih_pvPyr_scaling, gaba_pyr_scaling = gaba_pyr_scaling, iseed = iseed, wseed = wseed)
                    myfilename = '/u/mohdsh/Projects/ca3cannabisMTL/data/batch/{simstr}.h5py'.format(simstr = simstr)
                    if os.path.exists(myfilename): # skip if the simulation already exists
                        print (simstr + ' exists')
                        continue
                    NewParam(tmpsec,tmpopt,tmpval,'run','dorun',1)
                    NewParam(tmpsec,tmpopt,tmpval,'run','saveout',1)
                    NewParam(tmpsec,tmpopt,tmpval,'run','tstop',mytstop)
                    NewParam(tmpsec,tmpopt,tmpval,'run','simstr',simstr)
                    NewParam(tmpsec,tmpopt,tmpval,'net','scale',0)
                    NewParam(tmpsec,tmpopt,tmpval,'net','includeCCKcs',0)
                    NewParam(tmpsec,tmpopt,tmpval,'net','basPop_cNum', 200)
                    NewParam(tmpsec,tmpopt,tmpval,'net','olmPop_cNum', 200)
                    NewParam(tmpsec,tmpopt,tmpval,'net','olm_nmdarw_scaling', nmdarw_olm_scaling)
                    NewParam(tmpsec,tmpopt,tmpval,'stim','DoMakeSignal',0)
                    NewParam(tmpsec,tmpopt,tmpval,'seed','iseed',iseed)
                    NewParam(tmpsec,tmpopt,tmpval,'seed','wseed',wseed)
                    lsec.append(tmpsec); lopt.append(tmpopt); lval.append(tmpval); lconfigfilestr.append(simstr)
    return lsec, lopt, lval, lconfigfilestr
    
    batchRun(runBatch, './batchlog')

You can then plot the results of the simulations using the plotting code above after changing 'simstr' to match the simulation you want to plot.
