#!/bin/bash

# PKGBUILD edited by; Batuhan Başerdem <lastname.firstname@gmail.com>
# For MATLAB version 2020a

## This PKGBUILD creates an Arch Linux package for MATLAB
## In order to build the package the user must supply;
##      matlab.fik : Plain text file installation key
##      matlab.lic : License file
##      matlab.tar : Software tarball
## FILE INSTALLATION KEY & LICENSE FILE:
## https://www.mathworks.com/help/install/ug/install-using-a-file-installation-key.html
##      File installation key identifies this specific installation of matlab.
##      The license file authorizes that this key can use the toolboxes.
##      Go to License center on mathworks website;
##          https://www.mathworks.com/licensecenter
##      On install and activate tab; select (or create) an appropriate license
##      Navigate to download the license file and the file installation key
##      Download the license file and put the file in the repository
##      Copy paste the file installation key in a plain text file
## GETTING TARBALL
##      As of 2020a release; this is much easier.
##    !!On arch, to run the installer, libselinux is needed.
##      Download the matlab installer, after logging in and accepting license;
##          Advanced Options > I want to download without installing
##      Download the files to an empty directory called matlab
##      Run following;
##          tar --create --verbose --file matlab.tar .../matlab

# To perform partial install, set to true and modify the products list

pkgbase='matlab'
pkgname=(
    'matlab-licenses'
    'matlab-engine-for-python'
    'matlab-bin')
pkgver=9.8.0.1323502
pkgrel=1
pkgdesc='A high-level language for numerical computation and visualization'
arch=('x86_64')
url='http://www.mathworks.com'
license=(custom)
makedepends=(
    'gendesk'
    'python'
    'findutils')
# For 2020a; from https://hub.docker.com/r/mathworks/matlab-deps/dockerfile
depends=(
    'ca-certificates'
    'lsb-release'
    'alsa-lib'
    'atk'
    'libcap'
    'libcups'
    'libdbus'
    'fontconfig'
    'libgcrypt'
    'gdk-pixbuf2'
    'gst-plugins-base'
    'gstreamer'
    'gtk2'
    'krb5'
    'nspr'
    'nss'
    'pam'
    'pango'
    'cairo'
    'libselinux'
    'libsm'
    'libsndfile'
    'libx11'
    'libxcb'
    'libxcomposite'
    'libxcursor'
    'libxdamage'
    'libxext'
    'libxfixes'
    'libxft'
    'libxi'
    'libxmu'
    'libxrandr'
    'libxrender'
    'libxslt'
    'libxss'
    'libxt'
    'libxtst'
    'libxxf86vm'
    'procps-ng'
    'xorg-server-xvfb'
    'x11vnc'
    'sudo'
    'zlib')
# These I got from arch and afraid to play around
depends+=(
    'gcc6'
    'gconf'
    'glu'
    'gstreamer0.10-base'
    'libunwind'
    'libxp'
    'libxpm'
    'portaudio'
    'qt5-svg'
    'qt5-webkit'
    'qt5-websockets'
    'qt5-x11extras'
    'xerces-c')
source=("matlab.tar"
        "matlab.fik"
        "matlab.lic")
md5sums=("SKIP"
         "SKIP"
         "SKIP")

# Edit package build array for a installation with select toolkits
partialinstall=false
instdir="/opt/tmw/${pkgbase}"

prepare() {
    msg2 'Extracting file installation key'
    _fik=$(grep -o [0-9-]* ${pkgbase}.fik)

    msg2 'Modifying the installer settings'
    _set="${srcdir}/${pkgbase}/installer_input.txt"
    # Installation will be done to $srcdir/files
    sed -i "s|^# destinationFolder=|destinationFolder=${srcdir}/files|" "${_set}"
    sed -i "s|^# fileInstallationKey=|fileInstallationKey=${_fik}|"     "${_set}"
    sed -i "s|^# agreeToLicense=|agreeToLicense=yes|"                   "${_set}"
    sed -i "s|^# licensePath=|licensePath=${srcdir}/matlab.lic|"        "${_set}"
    # Select products if partialinstall is set
    if [ ! -z ${products+isSet} ]; then
        msg2 'Building a package with a subset of the licensed products.'
        for _prod in "${products[@]}"; do
            sed -i 's|^#\(product.'"${_prod}"'\)|\1|' "${_set}"
        done
    fi

    msg2 'Creating desktop file'
    gendesk -f -n \
        --pkgname "${pkgbase}" \
        --pkgdesc "${pkgdesc}" \
        --categories "Development;Education;Science;Mathematics;IDE" \
        --mimetypes "application/x-matlab-data;text/x-matlab" \
        --exec "${instdir}/matlab -desktop"
}

build() {
    msg2 'Starting MATLAB installer'
    # Using the installer with the -inputFile parameter will automatically
    #   cause the installation to be non-interactive
    "${srcdir}/${pkgbase}/install" -inputFile "${srcdir}/${pkgbase}/installer_input.txt"

    # https://aur.archlinux.org/packages/matlab-engine-for-python/
    # https://www.mathworks.com/help/matlab/matlab_external/install-the-matlab-engine-for-python.html
    msg2 'Installing matlab engine for python'
    cd "${srcdir}/files/extern/engines/python"
    # Getting appropriate python version for spoofing
    _prefix="$(python -c 'import sys; print(sys.prefix)')"
    _pytminor="$(python -c 'import sys; print(sys.version_info.minor)')"
    _matminor="$(find . -name 'matlabengineforpython3*.so' |
                 sort |
                 sed 's|.*matlabengineforpython3_\([0-9]\)\.so|\1|g' |
                 tail -1)"
    cat 'import sys' > "${srcdir}/sitecustomize.py"
    cat "sys.version_info = (3, ${_matminor}, 0)" >> "${srcdir}/sitecustomize.py"
    PYTHONPATH="${srcdir}" python setup.py
        build --build-base="${srcdir}" \
        install --root="${srcdir}" --optimize 1
    # Correct files if MATLAB is ancient (as always)
    if [[ "${_pytminor}" != "${_matminor}" ]]; then
        mv "${srcdir}/${_prefix}/lib/python3".{"${_matminor}","${_pytminor}"}
        _egginfo="$(ls "${srcdir}/${_prefix}/lib/python3.${_pytminor}/site-packages/"*"-py3.${_matminor}.egg-info")"
        mv "${_egginfo}" "${_egginfo%py3."${_matminor}".egg-info}py3.${_pytminor}.egg-info"
        sed -i "s|sys.version_info|(3, $mat_minor, 0)|" \
            "${srcdir}/${_prefix}/lib/python3.${_pytminor}/site-packages/matlab/engine/__init__.py"
    fi
}

package_matlab-licenses() {
    depends=("matlab-bin=$pkgver")
    mkdir -p "${pkgdir}/${instdir}"
    mv "${srcdir}/files/licenses" "${pkgdir}/${instdir}/licenses"
    install -D -m644 "${srcdir}/${pkgbase}/license_agreement.txt" "${pkgdir}/usr/share/licenses/${pkgbase}/LICENSE"
}

package_matlab-engine-for-python() {
    depends=("matlab-bin=$pkgver" 'python')
    
    # Get the used prefix
    _prefix="$(python -c 'import sys; print(sys.prefix)')"

    # Move to packaging files
    mv "${srcdir}/${_prefix}" "${pkgdir}"
    cd "${srcdir}/files/extern/engines/python"
}

package_matlab-bin() {
    msg2 'Moving files from staging area'
    mv "${srcdir}/files" "${pkgdir}/${instdir}"
    chown --recursive root:root "${pkgdir}/${instdir}"

    msg2 'Creating links for executables'
    install -d -m755 "${pkgdir}/usr/bin/"
    for _executable in deploytool matlab mbuild mcc; do
        ln -s "${instdir}/bin/${_executable}" "${pkgdir}/usr/bin/${_executable}"
    done
    ln -s "${instdir}/bin/mex" "${pkgdir}/usr/bin/mex-$pkgbase"

    msg2 'Installing desktop files'
    install -D -m644 "${pkgname}.desktop" "${pkgdir}/usr/share/applications/${pkgname}.desktop"

    msg2 'Creating backup directories'
    mkdir -p "${pkgdir}/${instdir}/backup/bin/glnxa64/mexopts"
    mkdir -p "${pkgdir}/${instdir}/backup/sys/os/glnxa64"

    msg2 'Configuring mex options'
    cp "${pkgdir}/${instdir}/bin/glnxa64/mexopts/gcc_glnxa64.xml" "${pkgdir}/${instdir}/backup/bin/glnxa64/mexopts/"
    sed -i "s/gcc/gcc-6/g" "${pkgdir}/${instdir}/bin/glnxa64/mexopts/gcc_glnxa64.xml"
    cp "${pkgdir}/${instdir}/bin/glnxa64/mexopts/g++_glnxa64.xml" "${pkgdir}/${instdir}/backup/bin/glnxa64/mexopts/"
    sed -i "s/g++/g++-6/g" "${pkgdir}/${instdir}/bin/glnxa64/mexopts/g++_glnxa64.xml"
    cp "${pkgdir}/${instdir}/bin/glnxa64/mexopts/gfortran.xml" "${pkgdir}/${instdir}/backup/bin/glnxa64/mexopts/"
    sed -i "s/gfortran/gfortran-6/g" "${pkgdir}/${instdir}/bin/glnxa64/mexopts/gfortran.xml"
    cp "${pkgdir}/${instdir}/bin/glnxa64/mexopts/gfortran6.xml" "${pkgdir}/${instdir}/backup/bin/glnxa64/mexopts/"
    sed -i "s/gfortran/gfortran-6/g" "${pkgdir}/${instdir}/bin/glnxa64/mexopts/gfortran6.xml"
    sed -i "s/gfortran6-/gfortran-6/g" "${pkgdir}/${instdir}/bin/glnxa64/mexopts/gfortran6.xml"

    # https://bbs.archlinux.org/viewtopic.php?id=236821
    msg2 'Removing unused library files'
    # See $MATLABROOT/sys/os/glnxa64/README.libstdc++
    mv "${pkgdir}/${instdir}/sys/os/glnxa64/"{libstdc++.so.6.0.22,libstdc++.so.6,libgcc_s.so.1,libgfortran.so.3.0.0,libgfortran.so.3,libquadmath.so.0.0.0,libquadmath.so.0} "${pkgdir}/${instdir}/backup/sys/os/glnxa64"
    # https://bbs.archlinux.org/viewtopic.php?id=236821
    mv "${pkgdir}/${instdir}/bin/glnxa64/"libfreetype.* "${pkgdir}/${instdir}/backup/bin/glnxa64"
    # make sure MATLAB can find libgfortran.so.3
    cp "${pkgdir}/${instdir}/bin/matlab" "${pkgdir}/${instdir}/backup/bin"
    sed -i 's,LD_LIBRARY_PATH="`eval echo $LD_LIBRARY_PATH`",LD_LIBRARY_PATH="`eval echo $LD_LIBRARY_PATH`:/usr/lib/gcc/x86_64-pc-linux-gnu/'$(pacman -Q gcc6 | awk '{print $2}' | cut -d- -f1)'",g' "${pkgdir}/${instdir}/bin/matlab"
}

if [ ! -z "${partialinstall+isSet}" ] && [ -z "${products+isSet}" ]; then
    products=(
        "5G_Toolbox"
        "AUTOSAR_Blockset"
        "Aerospace_Blockset"
        "Aerospace_Toolbox"
        "Antenna_Toolbox"
        "Audio_Toolbox"
        "Automated_Driving_Toolbox"
        "Bioinformatics_Toolbox"
        "Communications_Toolbox"
        "Computer_Vision_Toolbox"
        "Control_System_Toolbox"
        "Curve_Fitting_Toolbox"
        "DO_Qualification_Kit"
        "DSP_System_Toolbox"
        "Data_Acquisition_Toolbox"
        "Database_Toolbox"
        "Datafeed_Toolbox"
        "Deep_Learning_Toolbox"
        "Econometrics_Toolbox"
        "Embedded_Coder"
        "Filter_Design_HDL_Coder"
        "Financial_Instruments_Toolbox"
        "Financial_Toolbox"
        "Fixed_Point_Designer"
        "Fuzzy_Logic_Toolbox"
        "GPU_Coder"
        "Global_Optimization_Toolbox"
        "HDL_Coder"
        "HDL_Verifier"
        "IEC_Certification_Kit"
        "Image_Acquisition_Toolbox"
        "Image_Processing_Toolbox"
        "Instrument_Control_Toolbox"
        "LTE_Toolbox"
        "MATLAB"
        "MATLAB_Coder"
        "MATLAB_Compiler"
        "MATLAB_Compiler_SDK"
        "MATLAB_Parallel_Server"
        "MATLAB_Production_Server"
        "MATLAB_Report_Generator"
        "MATLAB_Web_App_Server"
        "Mapping_Toolbox"
        "Mixed_Signal_Blockset"
        "Model_Predictive_Control_Toolbox"
        "Model_Based_Calibration_Toolbox"
        "Motor_Control_Blockset"
        "Navigation_Toolbox"
        "OPC_Toolbox"
        "Optimization_Toolbox"
        "Parallel_Computing_Toolbox"
        "Partial_Differential_Equation_Toolbox"
        "Phased_Array_System_Toolbox"
        "Polyspace_Bug_Finder"
        "Polyspace_Bug_Finder_Server"
        "Polyspace_Code_Prover"
        "Polyspace_Code_Prover_Server"
        "Powertrain_Blockset"
        "Predictive_Maintenance_Toolbox"
        "RF_Blockset"
        "RF_Toolbox"
        "ROS_Toolbox"
        "Reinforcement_Learning_Toolbox"
        "Risk_Management_Toolbox"
        "Robotics_System_Toolbox"
        "Robust_Control_Toolbox"
        "Sensor_Fusion_and_Tracking_Toolbox"
        "SerDes_Toolbox"
        "Signal_Processing_Toolbox"
        "SimBiology"
        "SimEvents"
        "Simscape"
        "Simscape_Driveline"
        "Simscape_Electrical"
        "Simscape_Fluids"
        "Simscape_Multibody"
        "Simulink"
        "Simulink_3D_Animation"
        "Simulink_Check"
        "Simulink_Code_Inspector"
        "Simulink_Coder"
        "Simulink_Compiler"
        "Simulink_Control_Design"
        "Simulink_Coverage"
        "Simulink_Design_Optimization"
        "Simulink_Design_Verifier"
        "Simulink_Desktop_Real_Time"
        "Simulink_PLC_Coder"
        "Simulink_Real_Time"
        "Simulink_Report_Generator"
        "Simulink_Requirements"
        "Simulink_Test"
        "SoC_Blockset"
        "Spreadsheet_Link"
        "Stateflow"
        "Statistics_and_Machine_Learning_Toolbox"
        "Symbolic_Math_Toolbox"
        "System_Composer"
        "System_Identification_Toolbox"
        "Text_Analytics_Toolbox"
        "Trading_Toolbox"
        "Vehicle_Dynamics_Blockset"
        "Vehicle_Network_Toolbox"
        "Vision_HDL_Toolbox"
        "WLAN_Toolbox"
        "Wavelet_Toolbox"
        "Wireless_HDL_Toolbox"
    )
fi
