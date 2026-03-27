.custom-sponsor-icon {
    position: relative;
    width: 60px;           /* Adjust size as needed */
    height: 60px;
    border: 3px solid #4a90e2;   /* Blue border like in your image */
    border-radius: 12px;         /* Rounded corners */
    background: #f8f9fa;         /* Light background */
    overflow: hidden;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); /* Optional subtle shadow */
}

.person {
    position: relative;
    width: 32px;
    height: 38px;
    background: #2c3e50;         /* Dark color for the person silhouette */
    border-radius: 50% 50% 40% 40% / 60% 60% 40% 40%;  /* Head + shoulders shape */
}

.person::before {
    content: '';
    position: absolute;
    top: -12px;
    left: 50%;
    transform: translateX(-50%);
    width: 18px;
    height: 18px;
    background: #2c3e50;
    border-radius: 50%;          /* Head */
}

/* Blue medical cross in bottom-right */
.cross {
    position: absolute;
    bottom: 8px;
    right: 8px;
    width: 22px;
    height: 22px;
    background: #3498db;         /* Bright blue cross */
    border-radius: 4px;
}

.cross::before,
.cross::after {
    content: '';
    position: absolute;
    background: white;
    border-radius: 1px;
}

.cross::before {   /* Horizontal bar */
    top: 50%;
    left: 4px;
    right: 4px;
    height: 4px;
    transform: translateY(-50%);
}

.cross::after {    /* Vertical bar */
    left: 50%;
    top: 4px;
    bottom: 4px;
    width: 4px;
    transform: translateX(-50%);
}


<div class="custom-sponsor-icon" title="Sponsor Indicator">
    <div class="person"></div>
    <div class="cross"></div>
</div>
