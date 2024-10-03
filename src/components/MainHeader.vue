<template>
    <header>
        <div class="container">
            <div class="logo">
                <i class="bi bi-card-checklist"></i>
                <span>Listou</span>
            </div>
        </div>
    </header>
</template>
<script>
export default {
    name: 'MainHeader',
}
</script>
<style lang="css" scoped>
header {
    width: 100%;
    max-width: 500px;
    background-color: #F8772F;
    background-image: url('@/assets/header-background.png');
    background-position-x: right;
    background-repeat: no-repeat;
    background-size: cover;
    padding: 45px 0;
    display: table;
    margin: 0 auto;
}

.logo {
    font-size: 35px;
    font-weight: 600;
    color: #FFF;
    font-family: 'Neo Sans Std', sans-serif;
    display: flex;
    gap: 7px;
    align-items: center;
}

.logo i {
    font-size: 30px;
}
</style>