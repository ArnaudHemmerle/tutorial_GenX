import numpy as np
import h5py
import os as os

scan0 = np.genfromtxt('data/scan0.dat')
scan1 = np.genfromtxt('data/scan1.dat')
scan2 = np.genfromtxt('data/scan2.dat')

scans = [scan0, scan1, scan2]

# We change the upper bound of the electron density for each scan
max_rho = [0.04, 0.05, 0.045]

is_first_loop = True

for i, scan in enumerate(scans):
    
    f = h5py.File("data/model_for_fit.hgx", "a")
    
    ########################################################
    # Load the scan data into the model
    ########################################################
    f['current/data/datasets/0/x'][...] = scan[:,0]
    f['current/data/datasets/0/y'][...] = scan[:,1]
    f['current/data/datasets/0/error'][...] = scan[:,2]
    
    #########################################################
    # Change upper bound for electron density
    #########################################################
    # Index of columns (always the same)
    # 0: Parameter (string)
    # 1: Value (float)
    # 2: Fit (bool)
    # 3: Min (float)
    # 4: Max (float)
    # 5: Error (float)

    # Extract the parameter names
    col0 = [val.decode("UTF-8") for val in f['current/parameters/data col 0'].value]
    
    # Find where the param is
    arg_param = col0.index("Sub.setDens")
    # Change its max value (needs to extract and update the whole array)
    # 'Max' value is in data col 4 (see list above)
    temp = f['current/parameters/data col 4'].value
    temp[arg_param] = max_rho[i]
    f['current/parameters/data col 4'][...] = temp
    
    f.close()
    
    #########################################################
    # Do the fit
    #########################################################
    print(20*'#')
    print('Fit %s'%str(i))
    result_file = 'results_C/result'+str(i)+'.hgx'
    os.system('genx --run --mgen=100 data/model_for_fit.hgx '+result_file)
    
    #########################################################   
    # Extract and save the fit
    #########################################################
    
    f = h5py.File("results_C/result"+str(i)+".hgx", "r")

    # Extract and save the curves
    x = f['current/data/datasets/0/x'].value
    y = f['current/data/datasets/0/y'].value
    y_sim = f['current/data/datasets/0/y_sim'].value
    error = f['current/data/datasets/0/error'].value
 
    np.savetxt('results_C/fit'+str(i)+'.dat', np.transpose([x, y, y_sim, error]),
               header='#x\t#y\t#y_fit\t#error' )

    # Extract and save the parameters
    col0 = [val.decode("UTF-8") for val in f['current/parameters/data col 0'].value]
    col1 = [val for val in f['current/parameters/data col 1'].value]
    col2 = [val for val in f['current/parameters/data col 2'].value]
    col3 = [val for val in f['current/parameters/data col 3'].value]
    col4 = [val for val in f['current/parameters/data col 4'].value]
    col5 = [val for val in f['current/parameters/data col 5'].value]
    col_labels = [val.decode("UTF-8") for val in f['current/parameters/data_labels'].value]
    f.close()

    # Reorganise as a table and print
    tab0 = np.stack((col0, col1, col2, col3, col4, col5), axis = 1)
    tab = np.vstack([col_labels, tab0])
    print('Fit results')
    print(10*'-')
    for line in tab:
        if line[0] != '':
            try:
                print('%s\t%g'%(line[0], float(line[1])))
            except:
                print('%s\t%s'%(line[0], line[1]))
                
    print('')
    
    # Save
    if is_first_loop:
        # Write the header
        
        # Get rid of the empty lines in the parameter list
        pos_arg = np.array([True if val != '' else False for val in col0])
        
        # Mask the arrays to keep only the non-empty lines
        col0 = np.array(col0)[pos_arg]
        
        with open('results_C/summary.dat', 'w') as file:
            file.write('#index_fit\t')
            for item in col0:
                file.write('#%s\t'%item)
            file.write('\n')
        
        
        is_first_loop = False
    
    # Mask the arrays to keep only the non-empty lines
    col1 = np.array(col1)[pos_arg]
    with open('results_C/summary.dat', 'a') as file:
        file.write('%s\t'%str(i))
        for item in col1:
            file.write('%g\t'%item)
        file.write('\n')
    
    
       
    
    
    
    
    
    
    