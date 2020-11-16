<script>
export default {
  functional: true,
  render(createElement, context) {
    context.data.props = context.props;
    return createElement(
      require(`@/${context.props.data.template}`).default,
      context.data,
      context.children,
    );
  },
  props: ['data'],
};
</script>
